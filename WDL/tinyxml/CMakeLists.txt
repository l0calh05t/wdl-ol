add_library(
	tinyxml
	STATIC
	libxml_tinyxml.cpp libxml_tinyxml.h
	svgtiny_colors.c
	tinystr.cpp tinystr.h
	tinyxml.cpp tinyxml.h
	tinyxmlerror.cpp
	tinyxmlparser.cpp
)
target_include_directories(tinyxml INTERFACE .)
