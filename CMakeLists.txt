cmake_minimum_required(VERSION 3.9 FATAL_ERROR)
set(CMAKE_BUILD_TYPE_INIT Debug)
set(CMAKE_USER_MAKE_RULES_OVERRIDE IPlugDefaultFlags.cmake)
project(WDL-OL)

if(MSVC)
	add_definitions(
		-D_CRT_SECURE_NO_WARNINGS
		-D_CRT_NONSTDC_NO_DEPRECATE
	)
endif()

if(DEFINED CMAKE_BUILD_TYPE)
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS Debug Release MinSizeRel RelWithDebInfo)
endif()

add_subdirectory(WDL)

option(IPLUG_BUILD_EXAMPLES "Build IPlug example plugins" ON)
if(IPLUG_BUILD_EXAMPLES)
	add_subdirectory(IPlugExamples)
endif()
