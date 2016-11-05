for tangram-es the steps to add an external library would be:

add the submodule as you did in the example

include the CMakeLists.txt from sqlite like you did here https://github.com/hallahan/SQLiteCppDemo/blob/master/CMakeLists.txt#L6, in external/CMakeLists.txt

link core library to the external lib here https://github.com/tangrams/tangram-es/blob/master/core/CMakeLists.txt#L62

include the appropriate headers from the lib and add them to the include directories here https://github.com/tangrams/tangram-es/blob/master/core/CMakeLists.txt#L37
