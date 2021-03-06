function(buildSource SAMPLE_NAME)
	file(GLOB SAMPLE_SOURCES 
		${SAMPLE_NAME}/*.cpp 
		${SAMPLE_NAME}/*.h 
		${SAMPLE_NAME}/*.inl)
	file(GLOB SCENE_ELEMENTS_SOURCES 
		${SAMPLE_NAME}/SceneElements/*.cpp 
		${SAMPLE_NAME}/SceneElements/*.h 
		${SAMPLE_NAME}/SceneElements/*.inl)

	file(GLOB_RECURSE ALL_SHADER_SOURCES
		${SAMPLE_NAME}/Shaders/Common/*.vert
		${SAMPLE_NAME}/Shaders/Common/*.frag
		${SAMPLE_NAME}/Shaders/Common/*.geom
		${SAMPLE_NAME}/Shaders/Common/*.comp
		${SAMPLE_NAME}/Shaders/Common/*.tese
		${SAMPLE_NAME}/Shaders/Common/*.tesc 
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.vert
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.frag
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.geom
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.comp
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.tese
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.tesc
		${SAMPLE_NAME}/Shaders/RayTracing/*.rgen
		${SAMPLE_NAME}/Shaders/RayTracing/*.rchit
		${SAMPLE_NAME}/Shaders/RayTracing/*.rmiss )
	file(GLOB_RECURSE COMMON_SHADER_SOURCES
		${SAMPLE_NAME}/Shaders/Common/*.vert
		${SAMPLE_NAME}/Shaders/Common/*.frag
		${SAMPLE_NAME}/Shaders/Common/*.geom
		${SAMPLE_NAME}/Shaders/Common/*.comp
		${SAMPLE_NAME}/Shaders/Common/*.tese
		${SAMPLE_NAME}/Shaders/Common/*.tesc 
		${SAMPLE_NAME}/Shaders/Common/*.glsl)
	file(GLOB_RECURSE POSTPROCESS_SHADER_SOURCES
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.vert
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.frag
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.geom
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.comp
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.tese
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.tesc
		${SAMPLE_NAME}/Shaders/PostProcessEffects/*.glsl )
	file(GLOB_RECURSE RAYTRACING_SHADER_SOURCES
		${SAMPLE_NAME}/Shaders/RayTracing/*.rgen
		${SAMPLE_NAME}/Shaders/RayTracing/*.rchit
		${SAMPLE_NAME}/Shaders/RayTracing/*.rmiss
		${SAMPLE_NAME}/Shaders/RayTracing/*.glsl )

	file(GLOB UTILITY_SOURCES 
		${SAMPLE_NAME}/Utilities/*.cpp 
		${SAMPLE_NAME}/Utilities/*.h 
		${SAMPLE_NAME}/Utilities/*.inl)
	file(GLOB FORWARD_DECLARATION_SOURCES 
		${SAMPLE_NAME}/ForwardDeclaration/*.cpp 
		${SAMPLE_NAME}/ForwardDeclaration/*.h 
		${SAMPLE_NAME}/ForwardDeclaration/*.inl)

	file(GLOB VULKAN_SOURCES 
		${SAMPLE_NAME}/Vulkan/*.cpp 
		${SAMPLE_NAME}/Vulkan/*.h 
		${SAMPLE_NAME}/Vulkan/*.inl)
	file(GLOB VULKAN_RENDERER_BACKEND_SOURCES 
		${SAMPLE_NAME}/Vulkan/RendererBackend/*.cpp 
		${SAMPLE_NAME}/Vulkan/RendererBackend/*.h 
		${SAMPLE_NAME}/Vulkan/RendererBackend/*.inl)
	file(GLOB VULKAN_UTILITY_SOURCES 
		${SAMPLE_NAME}/Vulkan/Utilities/*.cpp 
		${SAMPLE_NAME}/Vulkan/Utilities/*.h 
		${SAMPLE_NAME}/Vulkan/Utilities/*.inl)

	
	SET(MAIN_CPP ${SAMPLE_NAME}/main.cpp)
	source_group("Source" FILES ${SAMPLE_SOURCES})
	source_group("Source\\SceneElements" FILES ${SCENE_ELEMENTS_SOURCES})

	source_group("Source\\Shaders" FILES ${COMMON_SHADER_SOURCES})
	source_group("Source\\Shaders\\PostProcessEffects" FILES ${POSTPROCESS_SHADER_SOURCES})
	source_group("Source\\Shaders\\RayTracing" FILES ${RAYTRACING_SHADER_SOURCES})
	
	source_group("Source\\ForwardDeclaration" FILES ${FORWARD_DECLARATION_SOURCES})
	source_group("Source\\Utilities" FILES ${UTILITY_SOURCES})

	source_group("Source\\Vulkan" FILES ${VULKAN_SOURCES})
	source_group("Source\\Vulkan\\Utilities" FILES ${VULKAN_UTILITY_SOURCES})
	source_group("Source\\Vulkan\\RendererBackend" FILES ${VULKAN_RENDERER_BACKEND_SOURCES})

	if(WIN32)
		add_executable(${SAMPLE_NAME} ${MAIN_CPP} 
			${SAMPLE_SOURCES}
			${SCENE_ELEMENTS_SOURCES}
			${ALL_SHADER_SOURCES}
			${RAYTRACING_SHADER_SOURCES}
			${FORWARD_DECLARATION_SOURCES} 	
			${UTILITY_SOURCES} 			
			${VULKAN_SOURCES} 
			${VULKAN_RENDERER_BACKEND_SOURCES} 
			${VULKAN_UTILITY_SOURCES})
		target_link_libraries(${SAMPLE_NAME} ${WINLIBS})
	else(WIN32)
		add_executable(${SAMPLE_NAME} ${MAIN_CPP} 
			${SAMPLE_SOURCES}
			${SCENE_ELEMENTS_SOURCES}
			${ALL_SHADER_SOURCES}
			${RAYTRACING_SHADER_SOURCES}
			${FORWARD_DECLARATION_SOURCES} 	
			${UTILITY_SOURCES} 			
			${VULKAN_SOURCES} 
			${VULKAN_RENDERER_BACKEND_SOURCES} 
			${VULKAN_UTILITY_SOURCES})
		target_link_libraries(${SAMPLE_NAME} ${CMAKE_THREAD_LIBS_INIT})
	endif(WIN32)

	foreach(SHADER_SOURCE ${ALL_SHADER_SOURCES})
		set(SHADER_DIR ${CMAKE_CURRENT_BINARY_DIR}/${SAMPLE_NAME}/Shaders)
		if(WIN32)
			get_filename_component(fname ${SHADER_SOURCE} NAME)
			add_custom_target(${SAMPLE_NAME}_${fname}.spv
							COMMAND ${CMAKE_COMMAND} -E make_directory ${SHADER_DIR} && 
							$ENV{VK_SDK_PATH}/Bin/glslangValidator.exe -V ${SHADER_SOURCE} -o ${SHADER_DIR}/${fname}.spv
							SOURCES ${SHADER_SOURCE} )
			ExternalTarget("Shaders" ${SAMPLE_NAME}_${fname}.spv)
			add_dependencies(${SAMPLE_NAME} ${SAMPLE_NAME}_${fname}.spv)
		endif(WIN32)
	endforeach()

	target_link_libraries(${SAMPLE_NAME} Vulkan::Vulkan glfw)
	target_include_directories( ${SAMPLE_NAME} PRIVATE
								${CMAKE_CURRENT_SOURCE_DIR} 
								${CMAKE_CURRENT_SOURCE_DIR}/${SAMPLE_NAME}
								${GLM_INCLUDE_DIR} )
			
	ExternalTarget("" ${SAMPLE_NAME})
endfunction(buildSource)

buildSource(MageFramework)