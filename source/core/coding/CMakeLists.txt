cmake_policy(SET CMP0076 NEW)
target_sources(open_htj2k
    PRIVATE
    coding_units.cpp
    ht_block_decoding.cpp
    ht_block_encoding.cpp
    block_decoding.cpp
    mq_decoder.cpp
)