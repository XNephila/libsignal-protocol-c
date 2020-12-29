if(BUILD_SHARED_LIBS)
	find_library(M_LIB m)
endif()

include_directories(
	.
	curve25519/ed25519/nacl_includes
	curve25519/ed25519/additions
	curve25519/ed25519/sha512
	curve25519/ed25519
	curve25519
)

set(protobuf_SRCS
	LocalStorageProtocol.pb-c.c
	WhisperTextProtocol.pb-c.c
	FingerprintProtocol.pb-c.c
)

set(signal_protocol_SRCS
	vpool.c
	vpool.h
	signal_protocol.c
	signal_protocol.h
	signal_protocol_types.h
	signal_protocol_internal.h
	curve.c
	curve.h
	hkdf.c
	hkdf.h
	ratchet.c
	ratchet.h
	protocol.c
	protocol.h
	session_state.c
	session_state.h
	session_record.c
	session_record.h
	session_pre_key.c
	session_pre_key.h
	session_builder.c
	session_builder.h
	session_builder_internal.h
	session_cipher.c
	session_cipher.h
	key_helper.c
	key_helper.h
	sender_key.c
	sender_key.h
	sender_key_state.c
	sender_key_state.h
	sender_key_record.c
	sender_key_record.h
	group_session_builder.c
	group_session_builder.h
	group_cipher.c
	group_cipher.h
	fingerprint.c
	fingerprint.h
	device_consistency.c
	device_consistency.h
)

add_subdirectory(curve25519)
add_subdirectory(protobuf-c)

add_library(signal-protocol-c
	${protobuf_SRCS}
	${signal_protocol_SRCS}
	$<TARGET_OBJECTS:curve25519>
	$<TARGET_OBJECTS:protobuf-c>
)

if(BUILD_SHARED_LIBS)
	target_link_libraries(signal-protocol-c ${M_LIB})
	set_target_properties(signal-protocol-c PROPERTIES
		VERSION ${SIGNAL_PROTOCOL_C_VERSION}
		SOVERSION ${SIGNAL_PROTOCOL_C_VERSION_MAJOR}
	)
endif()

INSTALL(
	FILES
	signal_protocol.h
	signal_protocol_types.h
	curve.h
	hkdf.h
	ratchet.h
	protocol.h
	session_state.h
	session_record.h
	session_pre_key.h
	session_builder.h
	session_cipher.h
	key_helper.h
	sender_key.h
	sender_key_state.h
	sender_key_record.h
	group_session_builder.h
	group_cipher.h
	fingerprint.h
	device_consistency.h
	DESTINATION ${INCLUDE_INSTALL_DIR}/signal
)

INSTALL(TARGETS signal-protocol-c EXPORT signal
	LIBRARY DESTINATION ${LIB_INSTALL_DIR}
	RUNTIME DESTINATION ${BIN_INSTALL_DIR}
	ARCHIVE DESTINATION ${LIB_INSTALL_DIR}
)

configure_file( ${CMAKE_CURRENT_SOURCE_DIR}/libsignal-protocol-c.pc.in
		${CMAKE_CURRENT_BINARY_DIR}/libsignal-protocol-c.pc @ONLY)

INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/libsignal-protocol-c.pc DESTINATION "${INSTALL_PKGCONFIG_DIR}")

set(project_config_in "${CMAKE_CURRENT_LIST_DIR}/libsignal-protocol-c.cmake.in")
set(project_config_out "${CMAKE_CURRENT_BINARY_DIR}/libsignal-protocol-c.cmake")
set(config_targets_file "libsignal-protocol-cConfigTargets.cmake")
set(export_dest_dir "${CMAKE_INSTALL_LIBDIR}/cmake/signal-protocol-c")
set(version_config_file "${CMAKE_CURRENT_BINARY_DIR}/libsignal-protocol-cConfigVersion.cmake")

install(EXPORT signal DESTINATION ${export_dest_dir} NAMESPACE signal:: FILE ${config_targets_file})

include(CMakePackageConfigHelpers)
configure_file("${project_config_in}" "${project_config_out}" @ONLY)

write_basic_package_version_file("${version_config_file}" VERSION ${SIGNAL_PROTOCOL_C_VERSION} COMPATIBILITY SameMajorVersion)
install(FILES "${project_config_out}" "${version_config_file}" DESTINATION "${export_dest_dir}")