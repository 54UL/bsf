# Find tool packages
if(BUILD_BSL)
	find_package(bison REQUIRED)
	find_package(flex REQUIRED)

	message(STATUS "Building BSL lexer/parser...")

	set(BS_FLEX_OPTIONS -L)
	set(BS_BISON_OPTIONS -l)

	if(WIN32)
		list(APPEND BS_FLEX_OPTIONS --wincompat)
	endif()

	execute_process(
			COMMAND ${flex_EXECUTABLE} ${BS_FLEX_OPTIONS} ${BSF_SOURCE_DIR}/Plugins/bsfSL/BsLexerFX.l
			COMMAND ${bison_EXECUTABLE} ${BS_BISON_OPTIONS} ${BSF_SOURCE_DIR}/Plugins/bsfSL/BsParserFX.y
			WORKING_DIRECTORY ${BSF_SOURCE_DIR}/Plugins/bsfSL
			RESULT_VARIABLE rv
	)

	file(RENAME "${BSF_SOURCE_DIR}/Plugins/bsfSL/BsLexerFX.h" "${BSF_SOURCE_DIR}/Plugins/bsfSL/BsLexerFX.h")
	file(RENAME "${BSF_SOURCE_DIR}/Plugins/bsfSL/BsLexerFX.c" "${BSF_SOURCE_DIR}/Plugins/bsfSL/BsLexerFX.c")
	file(RENAME "${BSF_SOURCE_DIR}/Plugins/bsfSL/BsParserFX.h" "${BSF_SOURCE_DIR}/Plugins/bsfSL/BsParserFX.h")
	file(RENAME "${BSF_SOURCE_DIR}/Plugins/bsfSL/BsParserFX.c" "${BSF_SOURCE_DIR}/Plugins/bsfSL/BsParserFX.c")
endif()

# Source files and their filters
include(CMakeSources.cmake)

# Find package
find_package(XShaderCompiler)

# Target
add_library(bsfSL SHARED ${BS_SL_SRC})

# Includes
target_include_directories(bsfSL PRIVATE "./")

# Defines
target_compile_definitions(bsfSL PRIVATE -DBS_SL_EXPORTS)

# Pre-build step
if(BUILD_BSL AND WIN32)
	add_custom_command(TARGET bsfSL PRE_BUILD
		COMMAND ${flex_EXECUTABLE} ${BS_FLEX_OPTIONS} ${BSF_SOURCE_DIR}/Plugins/bsfSL/BsLexerFX.l
		COMMAND ${bison_EXECUTABLE} ${BS_BISON_OPTIONS} ${BSF_SOURCE_DIR}/Plugins/bsfSL/BsParserFX.y
		COMMAND xcopy /Y /I BsLexerFX.h \"${BSF_SOURCE_DIR}/Plugins/bsfSL\"
		COMMAND xcopy /Y /I BsLexerFX.c \"${BSF_SOURCE_DIR}/Plugins/bsfSL\"
		COMMAND xcopy /Y /I BsParserFX.h \"${BSF_SOURCE_DIR}/Plugins/bsfSL\"
		COMMAND xcopy /Y /I BsParserFX.c \"${BSF_SOURCE_DIR}/Plugins/bsfSL\"
		COMMAND del BsLexerFX.h BsLexerFX.c BsParserFX.h BsParserFX.c
		COMMENT "Building BSL lexer/parser...")
endif()

# Libraries
## External lib: XShaderCompiler
target_link_libraries(bsfSL PRIVATE ${XShaderCompiler_LIBRARIES})

## Local libs
target_link_libraries(bsfSL PRIVATE bsf)

# IDE specific
set_property(TARGET bsfSL PROPERTY FOLDER Plugins)

# Install
install(
	TARGETS bsfSL
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
)

if(WIN32)
	install(
		FILES $<TARGET_PDB_FILE:bsfSL> 
		DESTINATION bin 
		OPTIONAL
	)
endif()