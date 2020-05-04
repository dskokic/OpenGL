OpenGL library comes by default with Linux, but for its operation it needs OS-native mechanism - for getting graphics context, drawing window, capture mouse events, etc.
There are several available libraries suitable for this purpose (GLUT, SDL, GLFW, ..), but after reading internet articles, GLFW seems the most suitable one and is regularly updated. Therefore GLFW library will be used in all examples to give OpenGL library needed context for operation.

It is best to get the GLFW library is source form and compile it locally - it is very easy, espacially when using CMake (which generates Makefile). Here are the steps how to build, setup and use GLFW library:

1. Download GLFW library source code (click on the button named "Source package") on  https://www.glfw.org/download.html
2. I downloaded it to $HOME/Downloads/glfw-3.3.2.zip. Unzip it here. This will create directory $HOME/Downloads/glfw-3.3.2/ in which all
   GLFW library source files are stored
2. Install CMake. Cmake is a tool which we'll use to compile GLFW libary: 
   > sudo apt install cmake
3. Change directory to the root directory of GLFW source tree (not the src subdirectory), e.g. to $HOME/Downloads/glfw-3.3.2/
   > cd $HOME/Downloads/glfw-3.3.2/
4. Run cmake to generate Makefile. If you would like to create dynamic GLFW library (glfw.so) instead of static (glfw.a) than you need to 
   tell it to cmake via command line option BUILD_SHARED_LIBS=ON
   > cmake -DBUILD_SHARED_LIBS=ON .

   This will result that compiled file(s) will be stored in the same directory where sources are stored (so called  IN-TRRE compilation). If you want 	 your binary files to be compiled in separate directory  then you need to adopt steps:
	- in step 3. you need to create this dir (>mkdir <build_glfw_dir>) 
		     and cd to it (>cd <build_glfw_dir>)
	- in step 4. you need to refer to this directory (absolutely or relatively) when calling cmake 
		     (> cmake -DBUILD_SHARED_LIBS=ON ..\<build_glfw_dir>)
	- you need to adopt subsequent references used in gcc -I -L -l options in subsequent steps. This intructions hereafter assume IN-TREE 		  compilation.
5. Now you run make to compile the library
   > make
   Note: if make fails it migt be necessary to install xorg-dev package (sudo apt install xorg-dev) and execute make again
5. So dynamic library is now created in $HOME/Downloads/glfw-3.3.2/src/libglfw.so  (IN-TREE compilation)
6. There is a single header file that we need to include in the application that uses GLFW library: 
   $HOME/Downloads/glfw-3.3.2/include/GLFW/glfw3.h

So, minimalistic example to check if header file is properly includable, would look like (test1.c):

	#include <GLFW/glfw3.h>

	int main(void) {
		
	}

7. When compiling you need to tell the gcc compiler:
	- the directories where it can find include (.h) files : by using -I option 
	- the directories where it can find dynamic library (.so) files : by using  -L option and 
	- which libraries to dynamically link to the application: by using -l option

   Here is how:

	> gcc -o test1 -I$HOME/Downloads/glfw-3.3.2/include -L$HOME/Downloads/glfw-3.3.2/src -lglfw -lGL  test1.c

8. When you try to run the application, dynamic loader (dl) will see that the application uses dynamic libraries, e.g. glfw among others, and will try 	  to load it. Dynamic loader doesn't know on its own where glfw library is, so we need to tell it. Linux dynamic loader by default searches libraries in standard predefined locations (/usr/lib ), which can be overriden by defining LD_LIBRARY_PATH environment variable which points to folder containing dynamic library. 
I see two ways to do make dynamic loader aware of this :
a) One safe way to do it is to set LD_LIBRARY_PATH only for execution of our application, like:

	> LD_LIBRARY_PATH=$HOME/Downloads/glfw-3.3.2/src ./test1

b) Other safe way is to  compile using special linker option (-Wl,-rpath=<path_to_dynamic_library>) :
	> gcc -o test1 -I$HOME/Downloads/glfw-3.3.2/include -L$HOME/Downloads/glfw-3.3.2/src -lglfw -lGL  test1.c

If we exported the variable (made it visible in shell environment to other applications) then it would affect other applications that use dynamic libraries placed in standard locations (setting LD_LIBRARY_PATH will override this)

9. As suggested on https://learnopengl.com/Getting-started/Creating-a-window, for users compiling with gcc, the following command line options may help you compile the project (it essentially specially whicg dynamic libraries to link to your application):
	-lglfw3 -lGL -lX11 -lpthread -lXrandr -lXi -ldl
Not correctly linking the corresponding libraries will generate many undefined reference errors. 


10. Customize and download  GLAD from https://glad.dav1d.de/. If I correctly understood, this is needed to map generic OpenGL header function definitions to concrete vendor's function implementation. Web page https://glad.dav1d.de/ is web service used to customize GLAD package we will download:
Choose: 
Language: c/C++
Specification:  OpenGL
API gl: Version 3.3
Profile: Core
Options: tick check box "Generate loader"

click generate. It generated glad.zip. Download it to $HOME/Downloads and unzip locally. It created folder $HOME/Downloads/glad with subfolders include and src.

11.  You need to 
	- add glad.c file to your project (to compile it with other source files) and 
	- add glad/include folders (glad and KHR) into your include(s) directoy (-I gcc option)

12. So, finally you are able to compile the source files containing 

	#include <glad/glad.h>
	#include <GLFW/glfw3.h>

	int main(void) {
		
	}

13. Just create the file e.g.  makeme with following content :

gcc $1.c $HOME/Downloads/glad/src/glad.c -o $1 -I$HOME/Downloads/glfw-3.3.2/include -I$HOME/Downloads/glad/include -L$HOME/Downloads/glfw-3.3.2/src -Wl,-rpath=$HOME/Downloads/glfw-3.3.2/src -lglfw -lX11 -lpthread -lXrandr -lXi -ldl

14. Make the file executable :

chmod a+x makeme

15. Compile your source file with makeme script (omit .c extension):

./makeme <filename without extension>

e.g.: ./makeme test1

16. Finally execute your program:

./test1

