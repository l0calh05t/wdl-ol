add_library(
	giflib
	STATIC
	config.h
	dgif_lib.c
	egif_lib.c
	gif_hash.c gif_hash.h
	gif_lib_private.h
	gif_lib.h
	gifalloc.c
)
target_include_directories(giflib INTERFACE .)
