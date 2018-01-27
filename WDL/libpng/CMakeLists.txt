add_library(
	libpng
	STATIC
	png.c png.h
	pngconf.h
	pngdebug.h
	pngerror.c
	pngget.c
	pnginfo.h
	pnglibconf.h
	pngmem.c
	pngread.c
	pngrio.c
	pngrtran.c
	pngrutil.c
	pngset.c
	pngstruct.h
	pngtest.c
	pngtrans.c
	pngwio.c
	pngwrite.c
	pngwtran.c
	pngwutil.c
)
target_include_directories(libpng INTERFACE .)
target_link_libraries(libpng PRIVATE zlib)
set_target_properties(libpng PROPERTIES FOLDER "Third Party")
