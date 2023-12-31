# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT license.

cmake_minimum_required(VERSION 3.16)

project(ccf_app LANGUAGES C CXX)

option(
  COMPILE_TARGET
  "Compile target to build for, one of [virtual;sgx;snp], defaults to virtual"
  virtual)

find_package("ccf_${COMPILE_TARGET}" REQUIRED)

add_ccf_app(ccf_app SRCS ${CMAKE_CURRENT_SOURCE_DIR}/app/app.cpp)

# Generate an ephemeral signing key
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/signing_key.pem
  COMMAND openssl genrsa -out ${CMAKE_CURRENT_BINARY_DIR}/signing_key.pem -3
  3072
)
add_custom_target(
  app_signing_key ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/signing_key.pem
)

sign_app_library(
  ccf_app.enclave ${CMAKE_CURRENT_SOURCE_DIR}/oe_sign.conf
  ${CMAKE_CURRENT_BINARY_DIR}/signing_key.pem
)