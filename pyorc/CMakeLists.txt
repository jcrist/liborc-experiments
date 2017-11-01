include_directories (SYSTEM "${ORC_PREFIX}/include")
add_executable (orc-scan FileScan.cc)
target_link_libraries (orc-scan liborc)
