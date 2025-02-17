#
# Copyright (C) 2011-2021 Intel Corporation. All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
#   * Redistributions of source code must retain the above copyright
#     notice, this list of conditions and the following disclaimer.
#   * Redistributions in binary form must reproduce the above copyright
#     notice, this list of conditions and the following disclaimer in
#     the documentation and/or other materials provided with the
#     distribution.
#   * Neither the name of Intel Corporation nor the names of its
#     contributors may be used to endorse or promote products derived
#     from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
#

include ../../../QuoteGeneration/buildenv.mk

WARM_Top_Path =../../../external/wasm-micro-runtime/

WARM_Lib_Path = $(WARM_Top_Path)/product-mini/platforms/linux/build/
JWT_CPP_Path = ../../../external/jwt-cpp/
RAPIDJSON_DIR =  ../../QVL/Src/ThirdParty/rapidjson/include
PREBUILD_OPENSSL_PATH  = ../../../prebuilt/openssl
QAL_Include_Path = -I./ \
				-I$(SGX_SDK)/include \
				-I$(PREBUILD_OPENSSL_PATH)/inc \
				-I$(WARM_Top_Path)/core/iwasm/include \
				-I$(JWT_CPP_Path)/include   \
				-I$(COMMON_DIR)/inc/internal \
				-I../../dcap_quoteverify/inc \
				-I$(RAPIDJSON_DIR)/

QAL_Cpp_Flags = $(CXXFLAGS) -g -fPIC $(QAL_Include_Path)
QAL_C_Flags = $(CFLAGS) -g -fPIC $(QAL_Include_Path)

QAL_Link_Flags := $(COMMON_LDFLAGS) -L$(WARM_Lib_Path) -lvmlib  -ldl -lm -lpthread \
				-L$(PREBUILD_OPENSSL_PATH)/lib/linux64 -lcrypto \
				-Wl,--gc-sections -Wl,--version-script=sgx_dcap_qal.lds

WASM_CONFIG = -DCMAKE_BUILD_TYPE=Release
ifeq ($(DEBUG), 1)
	WASM_CONFIG = -DCMAKE_BUILD_TYPE=Debug
	QAL_Link_Flags += -fsanitize=undefined
endif

ifeq ($(USE_LOCAL_WASM), 1)
	QAL_Cpp_Flags += -DUSE_LOCAL_WASM
endif

QAL_Cpp_Files := $(wildcard *.cpp)
QAL_CXX_Obj_Files = $(QAL_Cpp_Files:.cpp=.o)

QAL_C_Obj_Files = se_thread.o se_trace.o
ifdef QAL_BUILD
QAL_Obj_Files = $(QAL_CXX_Obj_Files) $(QAL_C_Obj_Files)
else
QVL_Cpp_Obj_Files = sgx_dcap_pcs_com.o config.o
QAL_Obj_Files = $(QAL_CXX_Obj_Files) $(QAL_C_Obj_Files)
endif
Target_Lib_Name = libsgx_dcap_qal.so

Target_Static_Lib_Name = libdcap_qal.a

.PHONY: all clean rebuild wasm_lib

all: $(Target_Lib_Name) $(Target_Static_Lib_Name) | $(BUILD_DIR)
	$(CP) $(Target_Lib_Name) $|

$(BUILD_DIR):
	@$(MKDIR) $@

$(Target_Lib_Name): $(QAL_Obj_Files) wasm_lib $(QVL_Cpp_Obj_Files)
	$(CXX) $(CXXFLAGS) $(QAL_Obj_Files) $(QVL_Cpp_Obj_Files) -shared -Wl,-soname=$@.$(SGX_MAJOR_VER) $(QAL_Link_Flags) -o $@
	ln -sf $(Target_Lib_Name) $(Target_Lib_Name).$(SGX_MAJOR_VER)

$(Target_Static_Lib_Name): $(QAL_Obj_Files) wasm_lib
	 $(CP) $(WARM_Lib_Path)/libvmlib.a $@
	 $(AR) rsD $@ $(QAL_Obj_Files)

sgx_dcap_pcs_com.o: %.o:../../dcap_quoteverify/%.cpp
	$(CXX) $(QAL_Cpp_Flags) -c $< -o $@
config.o: %.o:../../dcap_quoteverify/linux/%.cpp
	$(CXX) $(QAL_Cpp_Flags) -c $< -o $@

$(QAL_CXX_Obj_Files): %.o: %.cpp
	$(CXX) $(QAL_Cpp_Flags) -c $< -o $@

$(QAL_C_Obj_Files): %.o:$(COMMON_DIR)/src/%.c
	$(CC) $(QAL_C_Flags) -c $< -o $@

wasm_lib:
	test -f $(WARM_Lib_Path)/libvmlib.a || ($(MKDIR) $(WARM_Lib_Path) && cd $(WARM_Lib_Path) && cmake .. $(WASM_CONFIG) && $(MAKE) vmlib VERBOSE=1)


clean:
	$(RM) $(QAL_Obj_Files) $(Target_Lib_Name) $(Target_Lib_Name).$(SGX_MAJOR_VER) $(Target_Static_Lib_Name) $(BUILD_DIR)/$(Target_Lib_Name) $(QVL_Cpp_Obj_Files)
	$(RM) -r $(WARM_Lib_Path)

rebuild:
	$(MAKE)  clean
	$(MAKE)  all
