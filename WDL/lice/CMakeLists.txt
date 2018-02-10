add_library(
	lice
	STATIC
	lice_arc.cpp
	lice_bezier.h
	lice_bmp.cpp
	lice_colorspace.cpp
	lice_combine.h
	lice_extended.h
	lice_gif_write.cpp
	lice_gif.cpp
    # lice_gl_ctx.cpp lice_gl_ctx.h
    # lice_glbitmap.cpp lice_glbitmap.h
	lice_ico.cpp
	lice_image.cpp
	lice_jpg_write.cpp
	lice_lcf.cpp lice_lcf.h
	lice_line.cpp
	lice_lvg.cpp
	lice_palette.cpp
	lice_pcx.cpp
	lice_png_write.cpp
	lice_png.cpp
	lice_svg.cpp
	lice_texgen.cpp
	lice_text.cpp lice_text.h
	lice_textnew.cpp
	lice.cpp lice.h
)
target_include_directories(lice INTERFACE .)
target_link_libraries(lice PRIVATE giflib glew jpeglib libpng plush2 tinyxml zlib)
add_subdirectory(glew)
