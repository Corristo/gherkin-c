PROJECT(gherkin C)
cmake_minimum_required(VERSION 3.0)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

add_library(gherkin
        src/file_reader.c
        src/file_utf8_source.c
        src/print_utilities.c
        src/string_utilities.c
        src/unicode_utilities.c
        src/utf8_source.c
        src/parser.c
        src/token_scanner.c
        src/file_token_scanner.c
        src/string_token_scanner.c
        src/token_matcher.c
        src/token.c
        src/token_queue.c
        src/item_queue.c
        src/gherkin_line.c
        src/error.c
        src/error_list.c
        src/dialect.c
        src/ast_builder.c
        src/ast_node.c
        src/gherkin_document.c
        src/feature.c
        src/scenario.c
        src/scenario_definition.c
        src/background.c
        src/rule.c
        src/comment.c
        src/data_table.c
        src/doc_string.c
        src/example_table.c
        src/step.c
        src/table_cell.c
        src/table_row.c
        src/tag.c
        src/compiler.c
        src/pickle.c
        src/pickle_cell.c
        src/pickle_location.c
        src/pickle_row.c
        src/pickle_step.c
        src/pickle_string.c
        src/pickle_table.c
        src/pickle_tag.c
        src/event.c
        src/attachment_event.c
        src/gherkin_document_event.c
        src/pickle_event.c
        src/source_event.c
        src/ast_printer.c
        src/pickle_printer.c
    )

target_include_directories(gherkin
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    )

include(CheckSymbolExists)

check_symbol_exists(log10 math.h RES)
if(NOT RES)
  list(APPEND CMAKE_REQUIRED_LIBRARIES m)
  check_symbol_exists(log10 math.h RES2)
  if(RES2)
    target_link_libraries(gherkin PRIVATE m)
  else()
    message(FATAL_ERROR "No log10() found")
  endif()
endif()

add_executable(gherkinexe
        src/gherkin_cli.c
    )
target_link_libraries(gherkinexe PRIVATE gherkin)
target_include_directories(gherkinexe
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        ${CMAKE_CURRENT_SOURCE_DIR}/src
    )

add_executable(gherkin_generate_tokens
        src/token_formatter_builder.c
        src/gherkin_generate_tokens.c
    )
target_link_libraries(gherkin_generate_tokens PRIVATE gherkin)

include(CTest)
enable_testing()

file(GLOB GOOD_FEATURE_FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/testdata/good/*.feature
        )
file(GLOB BAD_FEATURE_FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/testdata/bad/*.feature
        )

foreach(ENTITY ${GOOD_FEATURE_FILES})
    get_filename_component(TEST_NAME ${ENTITY} NAME_WE)
    add_test(
        NAME ${TEST_NAME}
        COMMAND $<TARGET_FILE:gherkinexe> ${ENTITY}
    )
endforeach()

############ Installation section ############
set(include_install_dir "include")
set(lib_install_dir "lib/")
set(bin_install_dir "bin/")
set(config_install_dir "lib/cmake/")

set(project_config "${CMAKE_CURRENT_BINARY_DIR}/generated/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")

set(namespace "${PROJECT_NAME}::")

configure_file("${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in" "${project_config}" @ONLY)

# Install target (will install the library to specified CMAKE_INSTALL_PREFIX variable)
install(
  TARGETS gherkin
  EXPORT ${targets_export_name}
  ARCHIVE DESTINATION ${lib_install_dir}
  LIBRARY DESTINATION ${lib_install_dir}
  RUNTIME DESTINATION ${lib_install_dir}
  INCLUDES DESTINATION ${include_install_dir})

install(
  DIRECTORY "${PROJECT_SOURCE_DIR}/include/"
  DESTINATION ${include_install_dir}
  FILES_MATCHING PATTERN "*.*h")

install(
    FILES "${project_config}"
    DESTINATION "${config_install_dir}")

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}")

add_custom_target(install_${PROJECT_NAME}
        $(MAKE) install
        DEPENDS gherkin
        COMMENT "Installing ${PROJECT_NAME}")
