idf_component_register(SRCS "main.c" "llm.c"
                    INCLUDE_DIRS ""  LDFRAGMENTS "../linker.lf")


# Aggressive performance optimization flags for LLM inference
# Note: -Ofast and -ffast-math sacrifice IEEE 754 compliance for speed.
# This is acceptable for LLM inference where:
# 1. The model already uses approximations (fast_exp, fast_rsqrt, fast_sigmoid)
# 2. Input values are bounded (embeddings, attention scores)
# 3. Small numerical differences don't affect token selection quality
#
# If exact IEEE compliance is needed, replace -Ofast with -O3 and remove -ffast-math
target_compile_options(${COMPONENT_LIB} PRIVATE 
    -Ofast
    -ffast-math
    -funroll-loops
    -finline-functions
    -fomit-frame-pointer
    -ftree-vectorize
    -fno-if-conversion
)

# Create a SPIFFS image from the contents of the 'spiffs_image' directory
# that fits the partition named 'data'. FLASH_IN_PROJECT indicates that
# the generated image should be flashed when the entire project is flashed to
# the target with 'idf.py -p PORT flash'.
spiffs_create_partition_image(data ../data FLASH_IN_PROJECT)
