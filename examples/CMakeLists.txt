## zlib project
#
## LICENSE
# This source file is licensed under the zlib license. For the license 
# text, see the provided LICENSE.txt file, section 2.
# 
project(zlibTests C)
# this project entails "tests" and examples, because the zlib
# sources do not clearly distinguish between the two

## Targets ####################################################################
# example #
add_executable(example example.c)
target_link_libraries(example zlib::zlib)
add_test(example example)

# minigz #
add_executable(minigzip minigzip.c)
target_link_libraries(minigzip zlib::zlib)

if (HAVE_OFF64_T)
# example64 #
    add_executable(example64 example.c)
    target_link_libraries(example64 zlib::zlib zlib::z64)
    add_test(example64 example64)

# minigz #
    add_executable(minigzip64 minigzip.c)
    target_link_libraries(minigzip64 zlib::zlib zlib::z64)
endif ()

add_custom_command(TARGET example POST_BUILD
                   COMMAND "${CMAKE_COMMAND}" -E copy "$<TARGET_FILE:zlib::zlib>" "$<TARGET_FILE_DIR:example>"
                   )