add_subdirectory(giflib)
add_subdirectory(IPlug)
add_subdirectory(jpeglib)
add_subdirectory(libpng)
add_subdirectory(lice)
add_subdirectory(plush2)
add_subdirectory(tinyxml)
add_subdirectory(zlib)

add_library(
	WDL
	STATIC
	fft.c fft.h
	besselfilter.cpp besselfilter.h
	convoengine.cpp convoengine.h
)
target_include_directories(
	WDL
	INTERFACE
	.
)
