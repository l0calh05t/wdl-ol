set(IPLUG_SOURCES_COMMON
	IGraphics/IAutoGUI.h
	IGraphics/IControl.cpp IGraphics/IControl.h
	IGraphics/IControls.cpp IGraphics/IControls.h
	IGraphics/IGraphics.cpp IGraphics/IGraphics.h
	IGraphics/IGraphicsConstants.h
	IGraphics/IGraphicsLice.cpp IGraphics/IGraphicsLice.h
	IGraphics/IGraphicsLiveEdit.h
	IGraphics/IGraphicsStructs.h
	IGraphics/IGraphicsUtilites.h
	IGraphics/IKeyboardControl.h
	IGraphics/IPopupMenu.h
	IGraphics/NanoSVG/nanosvg.h
	IPlug_include_in_plug_hdr.h
	IPlug_include_in_plug_src.h
	IPlugBase.cpp IPlugBase.h
	IPlugBase_select.h
	IPlugBaseGraphics.cpp IPlugBaseGraphics.h
	IPlugConstants.h
	IPlugLogger.h
	IPlugMidi.h
	IPlugParameter.cpp IPlugParameter.h
	IPlugPlatform.h
	IPlugStructs.h
	IPlugUtilities.h
	NChanDelay.h
)

set(IPLUG_INCLUDE_DIRECTORIES_COMMON . IGraphics IGraphics/NanoSVG)

set(IPLUG_LINK_LIBRARIES_COMMON lice WDL)

if(WIN32)
	list(APPEND IPLUG_LINK_LIBRARIES_COMMON wininet comctl32)
	list(APPEND IPLUG_SOURCES_COMMON IGraphics/IGraphicsWin.cpp IGraphics/IGraphicsWin.h)
endif()

include(CMakeDependentOption)
set(AAX_AVAILABLE OFF)
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../AAX_SDK/Interfaces/AAX.h)
	message(STATUS "AAX SDK available")
	set(AAX_AVAILABLE ON)
endif()
cmake_dependent_option(IPLUG_AAX "Enable building AAX plugins" ON "AAX_AVAILABLE" OFF)

set(ASIO_AVAILABLE OFF)
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../ASIO_SDK/asio.h)
	message(STATUS "ASIO SDK available")
	set(ASIO_AVAILABLE ON)
endif()
cmake_dependent_option(IPLUG_ASIO "Enable building ASIO standalone applications" ON "ASIO_AVAILABLE" OFF)

# set(CA_AVAILABLE OFF)
# if(APPLE AND EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../CA_SDK/AudioUnits/AUPublic/AUBase/AUBase.h)
# 	message(STATUS "CoreAudio SDK available")
# 	set(CA_AVAILABLE ON)
# endif()
# cmake_dependent_option(IPLUG_CA "Enable building CA plugins" ON "CA_AVAILABLE" OFF)

# set(PT9_AVAILABLE OFF)
# if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../PT9_SDK/???)
# 	message(STATUS "PT9 SDK available")
# 	set(PT9_AVAILABLE ON)
# endif()
# cmake_dependent_option(IPLUG_PT9 "Enable building PT9 plugins" ON "PT9_AVAILABLE" OFF)

set(VST_AVAILABLE OFF)
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../VST_SDK/aeffectx.h)
	message(STATUS "VST SDK available")
	set(VST_AVAILABLE ON)
endif()
cmake_dependent_option(IPLUG_VST "Enable building VST plugins" ON "VST_AVAILABLE" OFF)

set(VST3_AVAILABLE OFF)
if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../VST3_SDK/pluginterfaces/vst/vsttypes.h)
	message(STATUS "VST3 SDK available")
	set(VST3_AVAILABLE ON)
endif()
cmake_dependent_option(IPLUG_VST3 "Enable building VST3 plugins" ON "VST3_AVAILABLE" OFF)

if(NOT (AAX_AVAILABLE OR ASIO_AVAILABLE OR CA_AVAILABLE OR PT9_AVAILABLE OR VST_AVAILABLE OR VST3_AVAILABLE)
	message(FATAL_ERROR "No valid audio SDKs found! Plaease make sure to install at least one SDK")
endif()

if(NOT (IPLUG_AAX OR IPLUG_ASIO OR IPLUG_CA OR IPLUG_PT9 OR IPLUG_VST OR IPLUG_VST3))
	message(FATAL_ERROR "No audio SDKs selected for build!")
endif()

if(IPLUG_VST)
	add_library(
		IPlug-vst
		STATIC
		IPlugVST.cpp IPlugVST.h
		../VST_SDK/aeffect.h
		../VST_SDK/aeffectx.h
		${IPLUG_SOURCES_COMMON}
	)
	target_link_libraries(IPlug-vst PUBLIC ${IPLUG_LINK_LIBRARIES_COMMON})
	target_compile_definitions(IPlug-vst PUBLIC VST_API)
	target_include_directories(IPlug-vst PUBLIC ../VST_SDK ${IPLUG_INCLUDE_DIRECTORIES_COMMON})
	set_target_properties(IPlug-vst PROPERTIES FOLDER IPlug)
endif()

if(IPLUG_VST3)
	set(VST_SDK TRUE)
	get_filename_component(SDK_ROOT ../VST3_SDK ABSOLUTE)
	add_subdirectory(${SDK_ROOT}/base VST3_SDK/base)
	target_include_directories(base PUBLIC ${SDK_ROOT})
	set_target_properties(base PROPERTIES FOLDER "VST3 SDK")
	add_subdirectory(${SDK_ROOT}/public.sdk VST3_SDK/public.sdk)
	target_include_directories(sdk PUBLIC ${SDK_ROOT})
	set_target_properties(sdk PROPERTIES FOLDER "VST3 SDK")

	add_library(
		IPlug-vst3
		STATIC
		IPlugVST3.cpp IPlugVST3.h
		${SDK_ROOT}/public.sdk/source/main/dllmain.cpp
		${SDK_ROOT}/public.sdk/source/vst/vstsinglecomponenteffect.cpp
		${IPLUG_SOURCES_COMMON}
	)
	target_link_libraries(IPlug-vst3 PUBLIC sdk ${IPLUG_LINK_LIBRARIES_COMMON})
	target_compile_definitions(IPlug-vst3 PUBLIC VST3_API)
	target_include_directories(IPlug-vst3 PUBLIC ${IPLUG_INCLUDE_DIRECTORIES_COMMON})
	set_target_properties(IPlug-vst3 PROPERTIES FOLDER IPlug)
	if(MSVC)
		target_sources(IPlug-vst3 INTERFACE ${SDK_ROOT}/public.sdk/source/main/winexport.def)
	endif()
endif()

# if(IPLUG_ASIO)
# 	add_library(
# 		IPlug-app
# 		STATIC
# 		IPlugAPP.cpp IPlugAPP.h
# 		rtaudiomidi/RtAudio.cpp rtaudiomidi/RtAudio.h
# 		rtaudiomidi/RtMidi.cpp rtaudiomidi/RtMidi.h
# 		../ASIO_SDK/asio.cpp ../ASIO_SDK/asio.h
# 		../ASIO_SDK/asiodrivers.cpp ../ASIO_SDK/asiodrivers.h
# 		../ASIO_SDK/asiolist.cpp ../ASIO_SDK/asiolist.h
# 		${IPLUG_SOURCES_COMMON}
# 	)
# 	target_link_libraries(IPlug-app PUBLIC sdk ${IPLUG_LINK_LIBRARIES_COMMON})
# 	target_compile_definitions(IPlug-app PUBLIC APP_API)
# 	target_include_directories(IPlug-app PUBLIC rtaudiomidi ../ASIO_SDK ${IPLUG_INCLUDE_DIRECTORIES_COMMON})
# 	set_target_properties(IPlug-app PROPERTIES FOLDER IPlug)
# endif()

include(CMakeParseArguments)
function(iplug_add_plugin PLUGIN_NAME)
	cmake_parse_arguments(PLUGIN "" "" "SOURCES;LIBRARIES" ${ARGN})
	foreach(ARG ${PLUGIN_UNPARSED_ARGUMENTS})
		message(FATAL_ERROR "Unkown argument \"${ARG}\" passed to iplug_add_plugin")
	endforeach()
	if(TARGET IPlug-vst)
		add_library(${PLUGIN_NAME}-vst MODULE ${PLUGIN_SOURCES})
		target_link_libraries(${PLUGIN_NAME}-vst PRIVATE IPlug-vst ${PLUGIN_LIBRARIES})
		set_target_properties(${PLUGIN_NAME}-vst PROPERTIES
			LIBRARY_OUTPUT_NAME ${PLUGIN_NAME}
			PDB_NAME ${PLUGIN_NAME}-vst
		)
		install(TARGETS ${PLUGIN_NAME}-vst DESTINATION .)
	endif()
	if(TARGET IPlug-vst3)
		add_library(${PLUGIN_NAME}-vst3 MODULE ${PLUGIN_SOURCES})
		target_link_libraries(${PLUGIN_NAME}-vst3 PRIVATE IPlug-vst3 ${PLUGIN_LIBRARIES})
		set_target_properties(${PLUGIN_NAME}-vst3 PROPERTIES
			LIBRARY_OUTPUT_NAME ${PLUGIN_NAME}
			PDB_NAME ${PLUGIN_NAME}-vst3
			SUFFIX .vst3
		)
		install(TARGETS ${PLUGIN_NAME}-vst3 DESTINATION .)
	endif()
	# if(TARGET IPlug-app)
	# 	add_executable(
	# 		${PLUGIN_NAME}-app
	# 		${PLUGIN_SOURCES}
	# 		app_wrapper/app_dialog.cpp
	# 		app_wrapper/app_main.cpp app_wrapper/app_main.h
	# 	)
	# 	target_link_libraries(${PLUGIN_NAME}-app PRIVATE IPlug-app ${PLUGIN_LIBRARIES})
	# 	set_target_properties(${PLUGIN_NAME}-app PROPERTIES
	# 		OUTPUT_NAME ${PLUGIN_NAME}
	# 		PDB_NAME ${PLUGIN_NAME}-app
	# 		WIN32_EXECUTABLE ON
	# 	)
	# 	install(TARGETS ${PLUGIN_NAME}-vst3 DESTINATION .)
	# endif()
endfunction()
