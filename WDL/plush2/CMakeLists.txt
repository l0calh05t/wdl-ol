add_library(
	plush2
	STATIC
	pl_cam.cpp
	pl_make.cpp
	pl_math.cpp
	pl_obj.cpp
	pl_pf_tex.h
	pl_putface.cpp
	pl_read_3ds.cpp
	pl_read_cob.cpp
	pl_read_jaw.cpp
	pl_spline.cpp
	plush.h
)
target_include_directories(plush2 INTERFACE .)
target_link_libraries(plush2 PUBLIC lice)
