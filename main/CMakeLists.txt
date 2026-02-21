idf_component_register(SRCS "main.c" "llm.c"
                    INCLUDE_DIRS ""  LDFRAGMENTS "../linker.lf")


# Aggressive performance optimization flags for LLM inference
# -Ofast: Enable all -O3 optimizations plus fast-math
# -ffast-math: Allow aggressive floating-point optimizations
# -funroll-loops: Unroll loops for better performance
# -finline-functions: Inline functions aggressively
# -fomit-frame-pointer: Free up a register for general use
# -ftree-vectorize: Enable auto-vectorization
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
