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
	target_compile_definitions(IPlugVST3 INTERFACE VST3_API)
	if(MSVC)
		target_sources(IPlugVST3 INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/IPlugVST3.def)
	endif()
endif()

include(CMakeParseArguments)
function(iplug_add_plugin PLUGIN_NAME)
	cmake_parse_arguments(PLUGIN "" "" "SOURCES;LIBRARIES" ${ARGN})
	foreach(ARG ${PLUGIN_UNPARSED_ARGUMENTS})
		message(FATAL_ERROR "Unkown argument \"${ARG}\" passed to iplug_add_plugin")
	endforeach()
	if(TARGET IPlugVST)
		add_library(${PLUGIN_NAME}-vst MODULE ${PLUGIN_SOURCES})
		target_link_libraries(${PLUGIN_NAME}-vst PRIVATE IPlugVST ${PLUGIN_LIBRARIES})
		set_target_properties(${PLUGIN_NAME}-vst PROPERTIES
			LIBRARY_OUTPUT_NAME ${PLUGIN_NAME}
			PDB_NAME ${PLUGIN_NAME}-vst
		)
		install(TARGETS ${PLUGIN_NAME}-vst DESTINATION .)
	endif()
	if(TARGET IPlugVST3)
		add_library(${PLUGIN_NAME}-vst3 MODULE ${PLUGIN_SOURCES})
		target_link_libraries(${PLUGIN_NAME}-vst3 PRIVATE IPlugVST3 ${PLUGIN_LIBRARIES})
		set_target_properties(${PLUGIN_NAME}-vst3 PROPERTIES
			LIBRARY_OUTPUT_NAME ${PLUGIN_NAME}
			PDB_NAME ${PLUGIN_NAME}-vst3
			SUFFIX .vst3
		)
		install(TARGETS ${PLUGIN_NAME}-vst3 DESTINATION .)
	endif()
endfunction()
