add_library(
	IPlug
	STATIC
	Containers.h
	Hosts.cpp Hosts.h
	IAutoGUI.h
	IBitmapMonoText.cpp
	IBitmapMonoText.h
	IControl.cpp IControl.h
	IKeyboardControl.h
	IMidiQueue.h
	IParam.cpp IParam.h
	IPlug_include_in_plug_hdr.h
	IPlug_include_in_plug_src.h
	IPlugBase.cpp IPlugBase.h
	IPlugOSDetect.h
	IPlugStructs.cpp IPlugStructs.h
	IPopupMenu.cpp IPopupMenu.h
	Log.cpp Log.h
	NChanDelay.h
)
target_include_directories(IPlug INTERFACE .)
target_link_libraries(IPlug PUBLIC lice WDL)

set(
	IPLUG_API_COMMON
	IGraphics.cpp IGraphics.h
)
if(WIN32)
	target_link_libraries(IPlug PRIVATE wininet comctl32)
	list(APPEND IPLUG_API_COMMON IGraphicsWin.cpp IGraphicsWin.h)
elseif(APPLE)
	find_library(CORE_GRAPHICS_LIBRARY CoreGraphics)
	find_library(CORE_FOUNDATION_LIBRARY CoreFoundation)
	find_library(APP_KIT_LIBRARY AppKit)
	find_library(CARBON_LIBRARY Carbon)
	target_link_libraries(IPlug PRIVATE ${CORE_GRAPHICS_LIBRARY} ${CORE_FOUNDATION_LIBRARY} ${APP_KIT_LIBRARY} ${CARBON_LIBRARY} swell)
	list(APPEND IPLUG_API_COMMON IGraphicsMac.mm IGraphicsMac.h IGraphicsCocoa.mm IGraphicsCocoa.h)
endif()

include(CMakeDependentOption)
set(AAX_AVAILABLE OFF)
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../../AAX_SDK/Interfaces/AAX.h)
	message(STATUS "AAX SDK available")
	set(AAX_AVAILABLE ON)
endif()
cmake_dependent_option(IPLUG_AAX "Enable building AAX plugins" ON "AAX_AVAILABLE" OFF)

set(ASIO_AVAILABLE OFF)
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../../ASIO_SDK/asio.h)
	message(STATUS "ASIO SDK available")
	set(ASIO_AVAILABLE ON)
endif()
cmake_dependent_option(IPLUG_ASIO "Enable building ASIO standalone applications" ON "ASIO_AVAILABLE" OFF)

# set(CA_AVAILABLE OFF)
# if(APPLE AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../../CA_SDK/AudioUnits/AUPublic/AUBase/AUBase.h)
# 	message(STATUS "CoreAudio SDK available")
# 	set(CA_AVAILABLE ON)
# endif()
# cmake_dependent_option(IPLUG_CA "Enable building CA plugins" ON "CA_AVAILABLE" OFF)

# set(PT9_AVAILABLE OFF)
# if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../../PT9_SDK/???)
# 	message(STATUS "PT9 SDK available")
# 	set(PT9_AVAILABLE ON)
# endif()
# cmake_dependent_option(IPLUG_PT9 "Enable building PT9 plugins" ON "PT9_AVAILABLE" OFF)

set(VST_AVAILABLE OFF)
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../../VST_SDK/aeffectx.h)
	message(STATUS "VST SDK available")
	set(VST_AVAILABLE ON)
endif()
cmake_dependent_option(IPLUG_VST "Enable building VST plugins" ON "VST_AVAILABLE" OFF)

set(VST3_AVAILABLE OFF)
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../../VST3_SDK/pluginterfaces/vst/vsttypes.h)
	message(STATUS "VST3 SDK available")
	set(VST3_AVAILABLE ON)
endif()
cmake_dependent_option(IPLUG_VST3 "Enable building VST3 plugins" ON "VST3_AVAILABLE" OFF)

if(IPLUG_VST)
	add_library(
		IPlugVST
		STATIC
		IPlugVST.cpp IPlugVST.h
		${IPLUG_API_COMMON}
	)
	target_link_libraries(IPlugVST PUBLIC IPlug)
	target_compile_definitions(IPlugVST INTERFACE VST_API)
endif()

if(IPLUG_VST3)
	set(VST_SDK TRUE)
	set(SDK_ROOT ../../VST3_SDK)
	add_subdirectory(../../VST3_SDK/base VST3_SDK/base)
	target_include_directories(base PUBLIC ../../VST3_SDK)
	add_subdirectory(../../VST3_SDK/public.sdk VST3_SDK/public.sdk)
	target_include_directories(sdk PUBLIC ../../VST3_SDK)

	add_library(
		IPlugVST3
		STATIC
		IPlugVST3.cpp IPlugVST3.h
		../../VST3_SDK/public.sdk/source/main/dllmain.cpp
		../../VST3_SDK/public.sdk/source/vst/vstsinglecomponenteffect.cpp
		${IPLUG_API_COMMON}
	)
	target_link_libraries(IPlugVST3 PUBLIC IPlug sdk)
	target_compile_definitions(IPlugVST3 INTERFACE VST3_API VST3_PRESET_LIST)
	if(MSVC)
		target_sources(IPlugVST3 INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/IPlugVST3.def)
	endif()
endif()

set(SWELL_RESGEN ${CMAKE_CURRENT_LIST_DIR}/../swell/mac_resgen.php CACHE INTERNAL "")

include(CMakeParseArguments)
function(iplug_add_plugin PLUGIN_NAME)
	cmake_parse_arguments(PLUGIN "" "" "SOURCES;LIBRARIES" ${ARGN})
	foreach(ARG ${PLUGIN_UNPARSED_ARGUMENTS})
		message(FATAL_ERROR "Unkown argument \"${ARG}\" passed to iplug_add_plugin")
	endforeach()
	if(TARGET IPlugVST)
		add_library(${PLUGIN_NAME}-vst MODULE ${PLUGIN_SOURCES})
        target_include_directories(${PLUGIN_NAME}-vst PRIVATE .)
		target_link_libraries(${PLUGIN_NAME}-vst PRIVATE IPlugVST ${PLUGIN_LIBRARIES})
		set_target_properties(${PLUGIN_NAME}-vst PROPERTIES
			OUTPUT_NAME ${PLUGIN_NAME}
			LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/VST2
			ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/VST2
			PREFIX ""
		)
		# this changes the extension of the .so to .vst... but maybe we should have a bundle/framework ending in .vst not a .so
		# if(APPLE)
		# 	set_target_properties(${PLUGIN_NAME}-vst PROPERTIES	SUFFIX ".vst")
		# endif()
		install(TARGETS ${PLUGIN_NAME}-vst DESTINATION .)
	endif()
	if(TARGET IPlugVST3)
		add_library(${PLUGIN_NAME}-vst3 MODULE ${PLUGIN_SOURCES})
        target_include_directories(${PLUGIN_NAME}-vst3 PRIVATE .)
		target_link_libraries(${PLUGIN_NAME}-vst3 PRIVATE IPlugVST3 ${PLUGIN_LIBRARIES})
		set_target_properties(${PLUGIN_NAME}-vst3 PROPERTIES
			OUTPUT_NAME ${PLUGIN_NAME}
			LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/VST3
			ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/VST3
			PREFIX ""
			SUFFIX .vst3
		)
		install(TARGETS ${PLUGIN_NAME}-vst3 DESTINATION .)
	endif()
	# this doesn't do what I expected
    # if(APPLE)
	# 	foreach(_TARGET ${PLUGIN_NAME}-vst ${PLUGIN_NAME}-vst3)
	# 		if(NOT TARGET ${_TARGET})
	# 			continue()
	# 		endif()
	# 		get_target_property(_SOURCES ${_TARGET} SOURCES)
	# 		foreach(_SRC ${_SOURCES})
	# 			if(NOT _SRC MATCHES "\.rc$")
	# 				continue()
	# 			endif()
	# 			get_filename_component(_BASE ${_SRC} NAME_WE)
	# 			get_filename_component(_ABS ${_SRC} ABSOLUTE)
	# 			get_filename_component(_DIR ${_ABS} DIRECTORY)
	# 			find_program(PHP_COMMAND php)
	# 			message("${PHP_COMMAND} ${SWELL_RESGEN} --force ${_SRC}")
	# 			message("${CMAKE_CURRENT_SOURCE_DIR}")
	# 			add_custom_command(
	# 				OUTPUT ${_DIR}/${_BASE}.rc_mac_dlg ${_DIR}/${_BASE}.rc_mac_menu
	# 				DEPENDS ${_SRC}
	# 				COMMAND ${PHP_COMMAND} ARGS ${SWELL_RESGEN} --force ${_SRC}
	# 				WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
	# 			)
	# 			target_sources(${_TARGET} PRIVATE ${_DIR}/${_BASE}.rc_mac_dlg ${_DIR}/${_BASE}.rc_mac_menu)
	# 		endforeach()
	# 	endforeach()
    # endif()
endfunction()
