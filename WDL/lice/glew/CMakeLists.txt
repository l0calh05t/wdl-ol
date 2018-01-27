add_library(
	glew
	STATIC
	include/GL/glew.h
	include/GL/glxew.h
	include/GL/wglew.h
	include/GL/WGLEXT.H
	src/glew.c
)
target_include_directories(glew INTERFACE include)
target_compile_definitions(glew PUBLIC GLEW_STATIC)
set_target_properties(glew PROPERTIES FOLDER "Third Party")
