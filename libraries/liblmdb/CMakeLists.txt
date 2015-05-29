cmake_minimum_required(VERSION 2.8)

add_library(lmdb STATIC mdb.c lmdb.h midl.c midl.h)
set_target_properties(lmdb PROPERTIES DEBUG_POSTFIX d)

install(TARGETS lmdb DESTINATION lib)

install(FILES getopt.h unistd.h lmdb.h midl.h DESTINATION include)

