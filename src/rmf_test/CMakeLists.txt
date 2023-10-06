cmake_minimum_required(VERSION 3.8)
project(rmf_test)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(gazebo_ros REQUIRED)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()

# ===== Code from TheConstruct tutorial: https://www.youtube.com/watch?v=POLjIOs2MaM&ab_channel=TheConstruct 
message(NOTICE "Starting compilation...")

file(GLOB_RECURSE traffic_editor_paths "rmf_config/*.building.yaml")

foreach(path ${traffic_editor_paths})
  message(NOTICE "path: ${path}")
  
  # Get the output world name
  string(REGEX REPLACE "\\.[^.]*\\.[^.]*$" "" no_extension_path ${path})
  string(REGEX MATCH "[^/]+$" world_name ${no_extension_path})

  set(map_path ${path})
  set(output_world_name ${world_name})
  set(output_dir ${CMAKE_CURRENT_BINARY_DIR}/maps/${output_world_name})
  set(output_world_path ${output_dir}/${output_world_name}.world)
  set(output_model_dir ${output_dir}/models)
  set(building_map_path ${map_path})
  
  message(NOTICE "map_path: ${map_path}")
  message(NOTICE "output_world_name: ${output_world_name}")
  message(NOTICE "output_dir: ${output_dir}")

  message(NOTICE "output_world_path: ${output_world_path}")
  message(NOTICE "output_model_dir: ${output_model_dir}")

  # Generate the Gazebo simulation world and download models
  message(NOTICE "Building gazebo world and downloading models...")

  message(NOTICE "EXECUTING COMMAND: ros2 run rmf_building_map_tools building_map_generator gazebo ${building_map_path} ${output_world_path} ${output_model_dir}")
  message(NOTICE "EXECUTING COMMAND: ros2 run rmf_building_map_tools building_map_model_downloader ${building_map_path} -f -e ~/.gazebo/models")
  
  add_custom_command(
    OUTPUT ${output_world_path}
    COMMAND ros2 run rmf_building_map_tools building_map_generator gazebo ${building_map_path} ${output_world_path} ${output_model_dir}
    COMMAND ros2 run rmf_building_map_tools building_map_model_downloader ${building_map_path} -f -e ~/.gazebo/models
    DEPENDS ${map_path}
  )

  add_custom_target(generate_${output_world_name}_world ALL
    DEPENDS ${output_world_path}
  )

  message(NOTICE "Done.")

  # Generate nav graphs
  set(output_nav_graphs_dir ${output_dir}/nav_graphs/)
  set(output_nav_graphs_phony ${output_dir}/nav_graphs/phony)

  message(NOTICE "Generating nav graphs in ${output_nav_graphs_dir}")
  message(NOTICE "EXECUTING COMMAND: ros2 run rmf_building_map_tools building_map_generator nav ${building_map_path} ${output_nav_graphs_dir}")

  add_custom_command(
    OUTPUT ${output_nav_graphs_phony}
    COMMAND ros2 run rmf_building_map_tools building_map_generator nav ${building_map_path} ${output_nav_graphs_dir}
    DEPENDS ${map_path}
  )

  add_custom_target(generate_${output_world_name}_nav_graphs ALL
    DEPENDS ${output_nav_graphs_phony}
  )

  install(
    DIRECTORY ${output_dir}
    DESTINATION share/${PROJECT_NAME}/maps
  )

  message(NOTICE "Generating lane graphs done!")

endforeach()

install(
  DIRECTORY
  rmf_config
  launch
  DESTINATION share/${PROJECT_NAME}
)

