# SLUS_206.71 — function symbols

The 7788 `STT_FUNC` symbols of the non-stripped PS2 ELF `SLUS_206.71`, read from its `.symtab`; each C++ name is demangled into `class` and `method`, with the mangled form kept in `raw_name`.

| vaddr | size | bind | class | method | raw_name |
|---|---|---|---|---|---|
| 0x001001e0 | 228 | GLOBAL |  | PS2_GetHeapMem__Fi | PS2_GetHeapMem__Fi |
| 0x001002d0 | 24 | GLOBAL |  | PS2_OutOfMem__Fv | PS2_OutOfMem__Fv |
| 0x001002f0 | 140 | LOCAL |  | largebin_index__FUi | largebin_index__FUi |
| 0x00100380 | 332 | LOCAL |  | malloc_init_state__FP12malloc_state | malloc_init_state__FP12malloc_state |
| 0x001004d0 | 988 | LOCAL |  | sYSMALLOc__FUiP12malloc_state | sYSMALLOc__FUiP12malloc_state |
| 0x001008b0 | 1272 | GLOBAL |  | malloc | malloc |
| 0x00100db0 | 352 | GLOBAL |  | free | free |
| 0x00100f10 | 320 | LOCAL |  | malloc_consolidate__FP12malloc_state | malloc_consolidate__FP12malloc_state |
| 0x00101050 | 696 | GLOBAL |  | realloc | realloc |
| 0x00101310 | 516 | GLOBAL |  | memalign | memalign |
| 0x00101520 | 388 | GLOBAL |  | mallinfo | mallinfo |
| 0x001016b0 | 36 | GLOBAL |  | mwInit | mwInit |
| 0x001016e0 | 8 | GLOBAL |  | mwExit | mwExit |
| 0x001016f0 | 156 | GLOBAL |  | __destroy_new_array | __destroy_new_array |
| 0x00101790 | 108 | GLOBAL |  | __destroy_arr | __destroy_arr |
| 0x00101800 | 292 | GLOBAL |  | __construct_array | __construct_array |
| 0x00101930 | 332 | GLOBAL |  | __construct_new_array | __construct_new_array |
| 0x00101a80 | 28 | GLOBAL |  | operator delete[] | __dla__FPv |
| 0x00101aa0 | 92 | GLOBAL |  | operator delete | __dl__FPv |
| 0x00101b00 | 76 | 13 | std::exception | {dtor} | __dt__Q23std9exceptionFv |
| 0x00101b50 | 12 | 13 | std::exception | what | what__Q23std9exceptionCFv |
| 0x00101b60 | 28 | GLOBAL |  | operator new[] | __nwa__FUi |
| 0x00101b80 | 124 | LOCAL | std | default_new_handler | default_new_handler__3stdFv |
| 0x00101c00 | 96 | 13 | std::bad_alloc | {dtor} | __dt__Q23std9bad_allocFv |
| 0x00101c60 | 12 | 13 | std::bad_alloc | what | what__Q23std9bad_allocCFv |
| 0x00101c70 | 172 | GLOBAL |  | operator new | __nw__FUi |
| 0x00101d20 | 612 | GLOBAL |  | __throw_catch_compare | __throw_catch_compare |
| 0x00101f90 | 36 | GLOBAL | std | unexpected | unexpected__3stdFv |
| 0x00101fc0 | 36 | GLOBAL | std | terminate | terminate__3stdFv |
| 0x00101ff0 | 36 | LOCAL | std | duhandler | duhandler__3stdFv |
| 0x00102020 | 28 | LOCAL | std | dthandler | dthandler__3stdFv |
| 0x00102040 | 68 | GLOBAL |  | __destroy_global_chain | __destroy_global_chain |
| 0x00102090 | 36 | GLOBAL |  | __register_global_object | __register_global_object |
| 0x001020c0 | 92 | GLOBAL |  | __initialize_cpp_rts | __initialize_cpp_rts |
| 0x00102120 | 160 | GLOBAL |  | __DecodeUnsignedNumber__FPcPUi | __DecodeUnsignedNumber__FPcPUi |
| 0x001021c0 | 160 | GLOBAL |  | __DecodeSignedNumber__FPcPi | __DecodeSignedNumber__FPcPi |
| 0x00102260 | 56 | GLOBAL |  | __end__catch | __end__catch |
| 0x001022a0 | 396 | GLOBAL |  | __ThrowHandler__FP12ThrowContext | __ThrowHandler__FP12ThrowContext |
| 0x00102430 | 804 | LOCAL |  | FindExceptionHandler__FP12ThrowContextP13ExceptionInfoPl | FindExceptionHandler__FP12ThrowContextP13ExceptionInfoPl |
| 0x00102760 | 452 | GLOBAL |  | __unexpected | __unexpected |
| 0x00102930 | 96 | 13 | std::bad_exception | {dtor} | __dt__Q23std13bad_exceptionFv |
| 0x00102990 | 372 | LOCAL |  | FindMostRecentException__FP12ThrowContextP13ExceptionInfo | FindMostRecentException__FP12ThrowContextP13ExceptionInfo |
| 0x00102b10 | 2148 | LOCAL |  | UnwindStack__FP12ThrowContextP13ExceptionInfoPc | UnwindStack__FP12ThrowContextP13ExceptionInfoPc |
| 0x00103380 | 788 | LOCAL |  | NextAction__FP14ActionIterator | NextAction__FP14ActionIterator |
| 0x001036a0 | 444 | LOCAL |  | FindExceptionRecord__FPcP13ExceptionInfo | FindExceptionRecord__FPcP13ExceptionInfo |
| 0x00103860 | 12 | 13 | std::bad_exception | what | what__Q23std13bad_exceptionCFv |
| 0x00103870 | 100 | GLOBAL |  | __TransferControl__FP12ThrowContextP13ExceptionInfoPc | __TransferControl__FP12ThrowContextP13ExceptionInfoPc |
| 0x001038e0 | 124 | GLOBAL |  | __throw | __throw |
| 0x00103960 | 76 | GLOBAL |  | __SkipUnwindInfo__FPc | __SkipUnwindInfo__FPc |
| 0x001039b0 | 32 | GLOBAL |  | __FindExceptionTable__FP13ExceptionInfoPc | __FindExceptionTable__FP13ExceptionInfoPc |
| 0x001039d0 | 176 | GLOBAL |  | __SetupFrameInfo__FP12ThrowContextP13ExceptionInfo | __SetupFrameInfo__FP12ThrowContextP13ExceptionInfo |
| 0x00103a80 | 656 | GLOBAL |  | __PopStackFrame__FP12ThrowContextP13ExceptionInfo | __PopStackFrame__FP12ThrowContextP13ExceptionInfo |
| 0x00103d10 | 124 | 13 | std::ios_base::failure | {dtor} | __dt__Q33std8ios_base7failureFv |
| 0x00103d90 | 84 | 13 | std::locale | {dtor} | __dt__Q23std6localeFv |
| 0x00103df0 | 208 | GLOBAL | std::ios_base | {dtor} | __dt__Q23std8ios_baseFv |
| 0x00103ec0 | 8 | 13 | std::basic_streambuf<w,Q23std14char_traits<w>> | setbuf | setbuf__Q23std41basic_streambuf<w,Q23std14char_traits<w>>FPwl |
| 0x00103ed0 | 8 | 13 | std::basic_streambuf<c,Q23std14char_traits<c>> | setbuf | setbuf__Q23std41basic_streambuf<c,Q23std14char_traits<c>>FPcl |
| 0x00103ee0 | 208 | GLOBAL | std::ios_base | init | init__Q23std8ios_baseFPv |
| 0x00103fb0 | 8 | 13 | std::ios_base::failure | what | what__Q33std8ios_base7failureCFv |
| 0x00103fc0 | 52 | WEAK | std::_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>> | {ctor} | __ct__Q23std80_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>>FRCQ23std80_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>> |
| 0x00104000 | 120 | WEAK | std::_RefCountedPtr<c,Q23std9_Array<c>> | {dtor} | __dt__Q23std34_RefCountedPtr<c,Q23std9_Array<c>>Fv |
| 0x00104080 | 136 | WEAK | std::_RefCountedPtr<c,Q23std9_Array<c>> | {ctor} | __ct__Q23std34_RefCountedPtr<c,Q23std9_Array<c>>FPc |
| 0x00104110 | 124 | WEAK | std::_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>> | {dtor} | __dt__Q23std80_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>>Fv |
| 0x00104190 | 320 | WEAK | std::__locale_imp<1> | {dtor} | __dt__Q23std15__locale_imp<1>Fv |
| 0x001042d0 | 76 | 13 | std::locale::facet | {dtor} | __dt__Q33std6locale5facetFv |
| 0x00104320 | 172 | WEAK | std::basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>> | {dtor} | __dt__Q23std59basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>>Fv |
| 0x001043d0 | 636 | GLOBAL | std::__nInit | {ctor} | __ct__Q23std7__nInitFv |
| 0x00104650 | 120 | 13 | std::basic_filebuf<c,Q23std14char_traits<c>> | {dtor} | __dt__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fv |
| 0x001046d0 | 224 | GLOBAL | std::__nInit | {dtor} | __dt__Q23std7__nInitFv |
| 0x001047b0 | 152 | 13 | std::basic_ostream<c,Q23std14char_traits<c>> | {dtor} | __dt__Q23std39basic_ostream<c,Q23std14char_traits<c>>Fv |
| 0x00104850 | 152 | 13 | std::basic_istream<c,Q23std14char_traits<c>> | {dtor} | __dt__Q23std39basic_istream<c,Q23std14char_traits<c>>Fv |
| 0x001048f0 | 636 | GLOBAL | std::__wInit | {ctor} | __ct__Q23std7__wInitFv |
| 0x00104b70 | 120 | 13 | std::basic_filebuf<w,Q23std14char_traits<w>> | {dtor} | __dt__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fv |
| 0x00104bf0 | 224 | GLOBAL | std::__wInit | {dtor} | __dt__Q23std7__wInitFv |
| 0x00104cd0 | 152 | 13 | std::basic_ostream<w,Q23std14char_traits<w>> | {dtor} | __dt__Q23std39basic_ostream<w,Q23std14char_traits<w>>Fv |
| 0x00104d70 | 152 | 13 | std::basic_istream<w,Q23std14char_traits<w>> | {dtor} | __dt__Q23std39basic_istream<w,Q23std14char_traits<w>>Fv |
| 0x00104e10 | 96 | 13 | std::basic_ios<w,Q23std14char_traits<w>> | {dtor} | __dt__Q23std35basic_ios<w,Q23std14char_traits<w>>Fv |
| 0x00104e70 | 104 | 13 | std::basic_streambuf<w,Q23std14char_traits<w>> | {dtor} | __dt__Q23std41basic_streambuf<w,Q23std14char_traits<w>>Fv |
| 0x00104ee0 | 96 | 13 | std::basic_ios<c,Q23std14char_traits<c>> | {dtor} | __dt__Q23std35basic_ios<c,Q23std14char_traits<c>>Fv |
| 0x00104f40 | 104 | 13 | std::basic_streambuf<c,Q23std14char_traits<c>> | {dtor} | __dt__Q23std41basic_streambuf<c,Q23std14char_traits<c>>Fv |
| 0x00104fb0 | 16 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | setbuf | setbuf__Q23std39basic_filebuf<w,Q23std14char_traits<w>>FPwl |
| 0x00104fc0 | 16 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | setbuf | setbuf__Q23std39basic_filebuf<c,Q23std14char_traits<c>>FPcl |
| 0x00104fd0 | 8 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | sync | sync__Q23std41basic_streambuf<w,Q23std14char_traits<w>>Fv |
| 0x00104fe0 | 8 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | showmanyc | showmanyc__Q23std41basic_streambuf<w,Q23std14char_traits<w>>Fv |
| 0x00104ff0 | 8 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | sync | sync__Q23std41basic_streambuf<c,Q23std14char_traits<c>>Fv |
| 0x00105000 | 8 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | showmanyc | showmanyc__Q23std41basic_streambuf<c,Q23std14char_traits<c>>Fv |
| 0x00105010 | 8 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | underflow | underflow__Q23std41basic_streambuf<c,Q23std14char_traits<c>>Fv |
| 0x00105020 | 84 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | uflow | uflow__Q23std41basic_streambuf<c,Q23std14char_traits<c>>Fv |
| 0x00105080 | 8 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | pbackfail | pbackfail__Q23std41basic_streambuf<c,Q23std14char_traits<c>>Fi |
| 0x00105090 | 8 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | overflow | overflow__Q23std41basic_streambuf<c,Q23std14char_traits<c>>Fi |
| 0x001050a0 | 292 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | xsputn | xsputn__Q23std41basic_streambuf<c,Q23std14char_traits<c>>FPCcl |
| 0x001051d0 | 284 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | xsgetn | xsgetn__Q23std41basic_streambuf<c,Q23std14char_traits<c>>FPcl |
| 0x001052f0 | 20 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | seekpos | seekpos__Q23std41basic_streambuf<c,Q23std14char_traits<c>>FQ23std7fpos<i>Q33std8ios_base8openmode |
| 0x00105310 | 20 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | seekoff | seekoff__Q23std41basic_streambuf<c,Q23std14char_traits<c>>FlQ33std8ios_base7seekdirQ33std8ios_base8openmode |
| 0x00105330 | 28 | WEAK | std::basic_streambuf<c,Q23std14char_traits<c>> | imbue | imbue__Q23std41basic_streambuf<c,Q23std14char_traits<c>>FRCQ23std6locale |
| 0x00105350 | 8 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | overflow | overflow__Q23std41basic_streambuf<w,Q23std14char_traits<w>>Fi |
| 0x00105360 | 476 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | xsputn | xsputn__Q23std41basic_streambuf<w,Q23std14char_traits<w>>FPCwl |
| 0x00105540 | 8 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | pbackfail | pbackfail__Q23std41basic_streambuf<w,Q23std14char_traits<w>>Fi |
| 0x00105550 | 84 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | uflow | uflow__Q23std41basic_streambuf<w,Q23std14char_traits<w>>Fv |
| 0x001055b0 | 8 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | underflow | underflow__Q23std41basic_streambuf<w,Q23std14char_traits<w>>Fv |
| 0x001055c0 | 468 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | xsgetn | xsgetn__Q23std41basic_streambuf<w,Q23std14char_traits<w>>FPwl |
| 0x001057a0 | 20 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | seekpos | seekpos__Q23std41basic_streambuf<w,Q23std14char_traits<w>>FQ23std7fpos<i>Q33std8ios_base8openmode |
| 0x001057c0 | 20 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | seekoff | seekoff__Q23std41basic_streambuf<w,Q23std14char_traits<w>>FlQ33std8ios_base7seekdirQ33std8ios_base8openmode |
| 0x001057e0 | 28 | WEAK | std::basic_streambuf<w,Q23std14char_traits<w>> | imbue | imbue__Q23std41basic_streambuf<w,Q23std14char_traits<w>>FRCQ23std6locale |
| 0x00105800 | 320 | WEAK | std::basic_istream<c,Q23std14char_traits<c>> | {ctor} | __ct__Q23std39basic_istream<c,Q23std14char_traits<c>>FPQ23std41basic_streambuf<c,Q23std14char_traits<c>> |
| 0x00105940 | 28 | 13 | std::ios_base | getloc | getloc__Q23std8ios_baseCFv |
| 0x00105960 | 316 | WEAK | std::basic_istream<w,Q23std14char_traits<w>> | {ctor} | __ct__Q23std39basic_istream<w,Q23std14char_traits<w>>FPQ23std41basic_streambuf<w,Q23std14char_traits<w>> |
| 0x00105aa0 | 12 | 13 | std::ctype<w> | do_widen | do_widen__Q23std8ctype<w>CFc |
| 0x00105ab0 | 476 | WEAK | std::basic_ostream<c,Q23std14char_traits<c>> | flush | flush__Q23std39basic_ostream<c,Q23std14char_traits<c>>Fv |
| 0x00105c90 | 312 | WEAK | std::basic_ostream<c,Q23std14char_traits<c>> | {ctor} | __ct__Q23std39basic_ostream<c,Q23std14char_traits<c>>FPQ23std41basic_streambuf<c,Q23std14char_traits<c>> |
| 0x00105dd0 | 476 | WEAK | std::basic_ostream<w,Q23std14char_traits<w>> | flush | flush__Q23std39basic_ostream<w,Q23std14char_traits<w>>Fv |
| 0x00105fb0 | 308 | WEAK | std::basic_ostream<w,Q23std14char_traits<w>> | {ctor} | __ct__Q23std39basic_ostream<w,Q23std14char_traits<w>>FPQ23std41basic_streambuf<w,Q23std14char_traits<w>> |
| 0x001060f0 | 232 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | imbue | imbue__Q23std39basic_filebuf<c,Q23std14char_traits<c>>FRCQ23std6locale |
| 0x001061e0 | 8 | 13 | std::codecvt<c,c,i> | do_always_noconv | do_always_noconv__Q23std14codecvt<c,c,i>CFv |
| 0x001061f0 | 28 | 13 | std::basic_streambuf<c,Q23std14char_traits<c>> | getloc | getloc__Q23std41basic_streambuf<c,Q23std14char_traits<c>>CFv |
| 0x00106210 | 172 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | sync | sync__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fv |
| 0x001062c0 | 456 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | seekpos | seekpos__Q23std39basic_filebuf<c,Q23std14char_traits<c>>FQ23std7fpos<i>Q33std8ios_base8openmode |
| 0x00106490 | 8 | 13 | std::codecvt<c,c,i> | do_encoding | do_encoding__Q23std14codecvt<c,c,i>CFv |
| 0x001064a0 | 612 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | seekoff | seekoff__Q23std39basic_filebuf<c,Q23std14char_traits<c>>FlQ33std8ios_base7seekdirQ33std8ios_base8openmode |
| 0x00106710 | 1100 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | overflow | overflow__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fi |
| 0x00106b60 | 204 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | pbackfail | pbackfail__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fi |
| 0x00106c30 | 424 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | uflow | uflow__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fv |
| 0x00106de0 | 496 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | underflow | underflow__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fv |
| 0x00106fd0 | 24 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | showmanyc | showmanyc__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fv |
| 0x00106ff0 | 352 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | {ctor} | __ct__Q23std39basic_filebuf<c,Q23std14char_traits<c>>FP7__sFILE |
| 0x00107150 | 232 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | imbue | imbue__Q23std39basic_filebuf<w,Q23std14char_traits<w>>FRCQ23std6locale |
| 0x00107240 | 8 | 13 | std::codecvt<w,c,i> | do_always_noconv | do_always_noconv__Q23std14codecvt<w,c,i>CFv |
| 0x00107250 | 28 | 13 | std::basic_streambuf<w,Q23std14char_traits<w>> | getloc | getloc__Q23std41basic_streambuf<w,Q23std14char_traits<w>>CFv |
| 0x00107270 | 172 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | sync | sync__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fv |
| 0x00107320 | 456 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | seekpos | seekpos__Q23std39basic_filebuf<w,Q23std14char_traits<w>>FQ23std7fpos<i>Q33std8ios_base8openmode |
| 0x001074f0 | 8 | 13 | std::codecvt<w,c,i> | do_encoding | do_encoding__Q23std14codecvt<w,c,i>CFv |
| 0x00107500 | 628 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | seekoff | seekoff__Q23std39basic_filebuf<w,Q23std14char_traits<w>>FlQ33std8ios_base7seekdirQ33std8ios_base8openmode |
| 0x00107780 | 1204 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | overflow | overflow__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fi |
| 0x00107c40 | 204 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | pbackfail | pbackfail__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fi |
| 0x00107d10 | 424 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | uflow | uflow__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fv |
| 0x00107ec0 | 512 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | underflow | underflow__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fv |
| 0x001080c0 | 40 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | showmanyc | showmanyc__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fv |
| 0x001080f0 | 352 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | {ctor} | __ct__Q23std39basic_filebuf<w,Q23std14char_traits<w>>FP7__sFILE |
| 0x00108250 | 148 | WEAK | std::_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>> | operator= | __as__Q23std80_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>>FRCQ23std80_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>> |
| 0x001082f0 | 180 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | read1 | read1__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fb |
| 0x001083b0 | 408 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | unshift | unshift__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fv |
| 0x00108550 | 180 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | read1 | read1__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fb |
| 0x00108610 | 396 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | read2 | read2__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fib |
| 0x001087a0 | 432 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | read3 | read3__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fb |
| 0x00108950 | 456 | WEAK | std::basic_filebuf<c,Q23std14char_traits<c>> | write2 | write2__Q23std39basic_filebuf<c,Q23std14char_traits<c>>Fic |
| 0x00108b20 | 220 | WEAK |  | __use_facet<Q23std8ctype<c>>__3stdFRCQ23std6localeQ23std12__chooser<1> | __use_facet<Q23std8ctype<c>>__3stdFRCQ23std6localeQ23std12__chooser<1> |
| 0x00108c00 | 212 | WEAK |  | __use_facet<Q23std8ctype<w>>__3stdFRCQ23std6localeQ23std12__chooser<1> | __use_facet<Q23std8ctype<w>>__3stdFRCQ23std6localeQ23std12__chooser<1> |
| 0x00108ce0 | 216 | WEAK |  | __use_facet<Q23std14codecvt<c,c,i>>__3stdFRCQ23std6localeQ23std12__chooser<1> | __use_facet<Q23std14codecvt<c,c,i>>__3stdFRCQ23std6localeQ23std12__chooser<1> |
| 0x00108dc0 | 216 | WEAK |  | __use_facet<Q23std14codecvt<w,c,i>>__3stdFRCQ23std6localeQ23std12__chooser<1> | __use_facet<Q23std14codecvt<w,c,i>>__3stdFRCQ23std6localeQ23std12__chooser<1> |
| 0x00108ea0 | 16 | 13 | std::codecvt<c,c,i> | do_out | do_out__Q23std14codecvt<c,c,i>CFRiPCcPCcRPCcPcPcRPc |
| 0x00108eb0 | 16 | 13 | std::codecvt<c,c,i> | do_in | do_in__Q23std14codecvt<c,c,i>CFRiPCcPCcRPCcPcPcRPc |
| 0x00108ec0 | 12 | 13 | std::codecvt<c,c,i> | do_unshift | do_unshift__Q23std14codecvt<c,c,i>CFRiPcPcRPc |
| 0x00108ed0 | 456 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | write2 | write2__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fiw |
| 0x001090a0 | 432 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | read3 | read3__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fb |
| 0x00109250 | 396 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | read2 | read2__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fib |
| 0x001093e0 | 408 | WEAK | std::basic_filebuf<w,Q23std14char_traits<w>> | unshift | unshift__Q23std39basic_filebuf<w,Q23std14char_traits<w>>Fv |
| 0x00109580 | 12 | 13 | std::codecvt<w,c,i> | do_unshift | do_unshift__Q23std14codecvt<w,c,i>CFRiPcPcRPc |
| 0x00109590 | 808 | WEAK | std::basic_ostream<c,Q23std14char_traits<c>>::sentry | {ctor} | __ct__Q33std39basic_ostream<c,Q23std14char_traits<c>>6sentryFRQ23std39basic_ostream<c,Q23std14char_traits<c>> |
| 0x001098c0 | 808 | WEAK | std::basic_ostream<w,Q23std14char_traits<w>>::sentry | {ctor} | __ct__Q33std39basic_ostream<w,Q23std14char_traits<w>>6sentryFRQ23std39basic_ostream<w,Q23std14char_traits<w>> |
| 0x00109bf0 | 8 | 13 | std::codecvt<c,c,i> | do_max_length | do_max_length__Q23std14codecvt<c,c,i>CFv |
| 0x00109c00 | 52 | 13 | std::codecvt<c,c,i> | do_length | do_length__Q23std14codecvt<c,c,i>CFRiPCcPCcUi |
| 0x00109c40 | 96 | 13 | std::codecvt<c,c,i> | {dtor} | __dt__Q23std14codecvt<c,c,i>Fv |
| 0x00109ca0 | 96 | WEAK | std::codecvt<w,c,i> | {dtor} | __dt__Q23std14codecvt<w,c,i>Fv |
| 0x00109d00 | 144 | WEAK | std::codecvt<w,c,i> | do_out | do_out__Q23std14codecvt<w,c,i>CFRiPCwPCwRPCwPcPcRPc |
| 0x00109d90 | 144 | WEAK | std::codecvt<w,c,i> | do_in | do_in__Q23std14codecvt<w,c,i>CFRiPCcPCcRPCcPwPwRPw |
| 0x00109e20 | 60 | WEAK | std::codecvt<w,c,i> | do_length | do_length__Q23std14codecvt<w,c,i>CFRiPCcPCcUi |
| 0x00109e60 | 8 | WEAK | std::codecvt<w,c,i> | do_max_length | do_max_length__Q23std14codecvt<w,c,i>CFv |
| 0x00109e70 | 620 | WEAK | std::ctype<w> | {ctor} | __ct__Q23std8ctype<w>FUi |
| 0x0010a0e0 | 264 | WEAK | std::__locale_imp<1> | replace | replace__Q23std15__locale_imp<1>FRCQ33std6locale5facetUi |
| 0x0010a1f0 | 212 | WEAK | std::ctype<w> | {dtor} | __dt__Q23std8ctype<w>Fv |
| 0x0010a2d0 | 64 | WEAK | std::ctype<w> | do_is | do_is__Q23std8ctype<w>CFUcw |
| 0x0010a310 | 112 | WEAK | std::ctype<w> | do_is | do_is__Q23std8ctype<w>CFPCwPCwPUc |
| 0x0010a380 | 128 | WEAK | std::ctype<w> | do_scan_is | do_scan_is__Q23std8ctype<w>CFUcPCwPCw |
| 0x0010a400 | 128 | WEAK | std::ctype<w> | do_scan_not | do_scan_not__Q23std8ctype<w>CFUcPCwPCw |
| 0x0010a480 | 36 | WEAK | std::ctype<w> | do_toupper | do_toupper__Q23std8ctype<w>CFw |
| 0x0010a4b0 | 100 | WEAK | std::ctype<w> | do_toupper | do_toupper__Q23std8ctype<w>CFPwPCw |
| 0x0010a520 | 36 | WEAK | std::ctype<w> | do_tolower | do_tolower__Q23std8ctype<w>CFw |
| 0x0010a550 | 100 | WEAK | std::ctype<w> | do_tolower | do_tolower__Q23std8ctype<w>CFPwPCw |
| 0x0010a5c0 | 56 | WEAK | std::ctype<w> | do_widen | do_widen__Q23std8ctype<w>CFPCcPCcPw |
| 0x0010a600 | 44 | WEAK | std::ctype<w> | do_narrow | do_narrow__Q23std8ctype<w>CFwc |
| 0x0010a630 | 140 | WEAK | std::ctype<w> | do_narrow | do_narrow__Q23std8ctype<w>CFPCwPCwcPc |
| 0x0010a6c0 | 100 | WEAK | std::__vector_pod<Ui,Q23std13allocator<Ui>> | resize | resize__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FUiRCUi |
| 0x0010a730 | 104 | WEAK | std::__vector_deleter<Q310Metrowerks15range_map<w,Uc>5entry,Q23std48allocator<Q310Metrowerks15range_map<w,Uc>5entry>> | {dtor} | __dt__Q23std112__vector_deleter<Q310Metrowerks15range_map<w,Uc>5entry,Q23std48allocator<Q310Metrowerks15range_map<w,Uc>5entry>>Fv |
| 0x0010a7a0 | 104 | WEAK | std::__vector_deleter<Q310Metrowerks14range_map<w,w>5entry,Q23std47allocator<Q310Metrowerks14range_map<w,w>5entry>> | {dtor} | __dt__Q23std110__vector_deleter<Q310Metrowerks14range_map<w,w>5entry,Q23std47allocator<Q310Metrowerks14range_map<w,w>5entry>>Fv |
| 0x0010a810 | 492 | WEAK | Metrowerks::range_map<w,Uc> | insert | insert__Q210Metrowerks15range_map<w,Uc>FRCwRCwRCUcRCUc |
| 0x0010aa00 | 468 | WEAK | Metrowerks::range_map<w,w> | insert | insert__Q210Metrowerks14range_map<w,w>FRCwRCwRCwRCw |
| 0x0010abe0 | 120 | WEAK | std | lower_bound<PQ310Metrowerks14range_map<w,w>5entry,Q310Metrowerks14range_map<w,w>5entry> | lower_bound<PQ310Metrowerks14range_map<w,w>5entry,Q310Metrowerks14range_map<w,w>5entry>__3stdFPQ310Metrowerks14range_map<w,w>5entryPQ310Metrowerks14range_map<w,w>5entryRCQ310Metrowerks14range_map<w,w>5entry |
| 0x0010ac60 | 144 | WEAK | std | lower_bound<PQ310Metrowerks15range_map<w,Uc>5entry,Q310Metrowerks15range_map<w,Uc>5entry> | lower_bound<PQ310Metrowerks15range_map<w,Uc>5entry,Q310Metrowerks15range_map<w,Uc>5entry>__3stdFPQ310Metrowerks15range_map<w,Uc>5entryPQ310Metrowerks15range_map<w,Uc>5entryRCQ310Metrowerks15range_map<w,Uc>5entry |
| 0x0010acf0 | 148 | WEAK | std::__vector_pod<Ui,Q23std13allocator<Ui>> | erase | erase__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FPUiPUi |
| 0x0010ad90 | 888 | WEAK | std::__vector_pod<Ui,Q23std13allocator<Ui>> | insert | insert__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FPUiUiRCUi |
| 0x0010b110 | 184 | 13 | std::length_error | {dtor} | __dt__Q23std12length_errorFv |
| 0x0010b1d0 | 168 | 13 | std::logic_error | {dtor} | __dt__Q23std11logic_errorFv |
| 0x0010b280 | 1496 | WEAK | std::__vector_imp<Q310Metrowerks15range_map<w,Uc>5entry,Q23std48allocator<Q310Metrowerks15range_map<w,Uc>5entry>,0> | insert | insert__Q23std110__vector_imp<Q310Metrowerks15range_map<w,Uc>5entry,Q23std48allocator<Q310Metrowerks15range_map<w,Uc>5entry>,0>FPQ310Metrowerks15range_map<w,Uc>5entryUiRCQ310Metrowerks15range_map<w,Uc>5entry |
| 0x0010b860 | 1480 | WEAK | std::__vector_imp<Q310Metrowerks14range_map<w,w>5entry,Q23std47allocator<Q310Metrowerks14range_map<w,w>5entry>,0> | insert | insert__Q23std108__vector_imp<Q310Metrowerks14range_map<w,w>5entry,Q23std47allocator<Q310Metrowerks14range_map<w,w>5entry>,0>FPQ310Metrowerks14range_map<w,w>5entryUiRCQ310Metrowerks14range_map<w,w>5entry |
| 0x0010be30 | 80 | WEAK | std::__vector_deleter<Q310Metrowerks15range_map<w,Uc>5entry,Q23std48allocator<Q310Metrowerks15range_map<w,Uc>5entry>> | clear | clear__Q23std112__vector_deleter<Q310Metrowerks15range_map<w,Uc>5entry,Q23std48allocator<Q310Metrowerks15range_map<w,Uc>5entry>>Fv |
| 0x0010be80 | 72 | WEAK | std::__vector_deleter<Q310Metrowerks14range_map<w,w>5entry,Q23std47allocator<Q310Metrowerks14range_map<w,w>5entry>> | clear | clear__Q23std110__vector_deleter<Q310Metrowerks14range_map<w,w>5entry,Q23std47allocator<Q310Metrowerks14range_map<w,w>5entry>>Fv |
| 0x0010bed0 | 184 | WEAK | Metrowerks::range_map<w,Uc> | operator[] | __vc__Q210Metrowerks15range_map<w,Uc>CFRCw |
| 0x0010bf90 | 168 | WEAK | Metrowerks::range_map<w,w> | operator[] | __vc__Q210Metrowerks14range_map<w,w>CFRCw |
| 0x0010c040 | 8 | 13 | std::logic_error | what | what__Q23std11logic_errorCFv |
| 0x0010c050 | 120 | WEAK | std | lower_bound<PCQ310Metrowerks14range_map<w,w>5entry,w> | lower_bound<PCQ310Metrowerks14range_map<w,w>5entry,w>__3stdFPCQ310Metrowerks14range_map<w,w>5entryPCQ310Metrowerks14range_map<w,w>5entryRCw |
| 0x0010c0d0 | 144 | WEAK | std | lower_bound<PCQ310Metrowerks15range_map<w,Uc>5entry,w> | lower_bound<PCQ310Metrowerks15range_map<w,Uc>5entry,w>__3stdFPCQ310Metrowerks15range_map<w,Uc>5entryPCQ310Metrowerks15range_map<w,Uc>5entryRCw |
| 0x0010c160 | 16 | 13 |  | @8@72@__dt__Q23std39basic_ostream<c,Q23std14char_traits<c>>Fv | @8@72@__dt__Q23std39basic_ostream<c,Q23std14char_traits<c>>Fv |
| 0x0010c170 | 16 | 13 |  | @24@72@__dt__Q23std39basic_istream<c,Q23std14char_traits<c>>Fv | @24@72@__dt__Q23std39basic_istream<c,Q23std14char_traits<c>>Fv |
| 0x0010c180 | 16 | 13 |  | @8@72@__dt__Q23std39basic_ostream<w,Q23std14char_traits<w>>Fv | @8@72@__dt__Q23std39basic_ostream<w,Q23std14char_traits<w>>Fv |
| 0x0010c190 | 16 | 13 |  | @24@72@__dt__Q23std39basic_istream<w,Q23std14char_traits<w>>Fv | @24@72@__dt__Q23std39basic_istream<w,Q23std14char_traits<w>>Fv |
| 0x0010c1a0 | 148 | GLOBAL | std::locale | global | global__Q23std6localeFv |
| 0x0010c240 | 224 | GLOBAL | std::locale | classic | classic__Q23std6localeFv |
| 0x0010c320 | 96 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | {dtor} | __dt__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>Fv |
| 0x0010c380 | 100 | GLOBAL | std::ctype<c> | {ctor} | __ct__Q23std8ctype<c>FPCUcbUi |
| 0x0010c3f0 | 128 | GLOBAL | std::ctype<c> | {dtor} | __dt__Q23std8ctype<c>Fv |
| 0x0010c470 | 20 | GLOBAL | std::ctype<c> | do_toupper | do_toupper__Q23std8ctype<c>CFc |
| 0x0010c490 | 56 | GLOBAL | std::ctype<c> | do_toupper | do_toupper__Q23std8ctype<c>CFPcPCc |
| 0x0010c4d0 | 20 | GLOBAL | std::ctype<c> | do_tolower | do_tolower__Q23std8ctype<c>CFc |
| 0x0010c4f0 | 56 | GLOBAL | std::ctype<c> | do_tolower | do_tolower__Q23std8ctype<c>CFPcPCc |
| 0x0010c530 | 8 | GLOBAL | std::ctype<c> | do_widen | do_widen__Q23std8ctype<c>CFc |
| 0x0010c540 | 48 | GLOBAL | std::ctype<c> | do_widen | do_widen__Q23std8ctype<c>CFPCcPCcPc |
| 0x0010c570 | 8 | GLOBAL | std::ctype<c> | do_narrow | do_narrow__Q23std8ctype<c>CFcc |
| 0x0010c580 | 48 | GLOBAL | std::ctype<c> | do_narrow | do_narrow__Q23std8ctype<c>CFPCcPCccPc |
| 0x0010c5b0 | 140 | WEAK | std::_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>> | {ctor} | __ct__Q23std80_RefCountedPtr<Q23std15__locale_imp<1>,Q23std32_Single<Q23std15__locale_imp<1>>>FPQ23std15__locale_imp<1> |
| 0x0010c640 | 92 | WEAK | std::basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>> | assign | assign__Q23std59basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>>FPCcUi |
| 0x0010c6a0 | 296 | WEAK | std::basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>> | {ctor} | __ct__Q23std59basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>>FPCcRCQ23std12allocator<c> |
| 0x0010c7d0 | 208 | WEAK | std::basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>> | {ctor} | __ct__Q23std59basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>>FRCQ23std12allocator<c> |
| 0x0010c8a0 | 256 | WEAK | std::basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>> | init | init__Q23std59basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>>FPCcPCcRCQ23std12allocator<c>Q23std26random_access_iterator_tag |
| 0x0010c9a0 | 916 | WEAK | std::basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>> | change_size | change_size__Q23std59basic_string<c,Q23std14char_traits<c>,Q23std12allocator<c>>FUib |
| 0x0010cd40 | 116 | GLOBAL | C_cache_base_block | {ctor} | __ct__18C_cache_base_blockFv |
| 0x0010cdc0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std30allocator<17S_cache_model_set>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details64compressed_pair_imp<Q23std30allocator<17S_cache_model_set>,Ui,1>FUi |
| 0x0010cdd0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18S_cache_base_block>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_cache_base_block>,Ui,1>FUi |
| 0x0010cde0 | 140 | GLOBAL | C_cache_base_block | {dtor} | __dt__18C_cache_base_blockFv |
| 0x0010ce70 | 424 | GLOBAL | C_cache_base_block | Close | Close__18C_cache_base_blockFv |
| 0x0010d020 | 64 | GLOBAL | C_cache_base_block | PrepareScene | PrepareScene__18C_cache_base_blockFv |
| 0x0010d060 | 372 | 13 | C_cache_base_block::S_hlp$2095_m_cache_cpp | cbDupl | cbDupl__Q218C_cache_base_block22S_hlp$2095_m_cache_cppFP9I3D_frameUi |
| 0x0010d1e0 | 5276 | GLOBAL | C_cache_base_block | Open | Open__18C_cache_base_blockFP7C_chunkP9I3D_scene |
| 0x0010e680 | 104 | 13 | S_cache_model_set | {dtor} | __dt__17S_cache_model_setFv |
| 0x0010e6f0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std30allocator<17S_cache_model_set>,Ui,1> | first | first__Q310Metrowerks7details64compressed_pair_imp<Q23std30allocator<17S_cache_model_set>,Ui,1>Fv |
| 0x0010e700 | 8 | 13 | std::__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>> | data | data__Q23std76__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>>CFv |
| 0x0010e710 | 84 | 13 | std::__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0> | {dtor} | __dt__Q23std76__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0>Fv |
| 0x0010e770 | 20 | 13 | std::__vector_deleter<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>> | {ctor} | __ct__Q23std78__vector_deleter<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>>Fv |
| 0x0010e790 | 140 | 13 | std::vector<P9I3D_frame,Q23std22allocator<P9I3D_frame>> | {dtor} | __dt__Q23std50vector<P9I3D_frame,Q23std22allocator<P9I3D_frame>>Fv |
| 0x0010e820 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std13allocator<Ui>,Ui,1> | first | first__Q310Metrowerks7details47compressed_pair_imp<Q23std13allocator<Ui>,Ui,1>Fv |
| 0x0010e830 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std13allocator<Ui>,Ui,1> | second | second__Q310Metrowerks7details47compressed_pair_imp<Q23std13allocator<Ui>,Ui,1>Fv |
| 0x0010e840 | 104 | 13 | std::__vector_imp<P9I3D_frame,Q23std22allocator<P9I3D_frame>,1> | {dtor} | __dt__Q23std58__vector_imp<P9I3D_frame,Q23std22allocator<P9I3D_frame>,1>Fv |
| 0x0010e8b0 | 20 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | {ctor} | __ct__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>Fv |
| 0x0010e8d0 | 420 | GLOBAL | C_cache_base_block | AssociateBaseBlocks | AssociateBaseBlocks__18C_cache_base_blockFv |
| 0x0010ea80 | 8 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | data | data__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>Fv |
| 0x0010ea90 | 168 | GLOBAL | C_cache_base_block | GetNumModelVisual | GetNumModelVisual__18C_cache_base_blockFP9I3D_modelPP10I3D_object |
| 0x0010eb40 | 276 | 13 | C_cache_base_block::S_hlp$2923_m_cache_cpp | cbVisCount | cbVisCount__Q218C_cache_base_block22S_hlp$2923_m_cache_cppFP9I3D_framePiPP10I3D_object |
| 0x0010ec60 | 1844 | GLOBAL | C_cache_base_block | OpenModel | OpenModel__18C_cache_base_blockFPCc |
| 0x0010f3a0 | 112 | 13 | C_string | {dtor} | __dt__8C_stringFv |
| 0x0010f410 | 252 | 13 | C_fstring | {ctor} | __ct__9C_fstringFPCce |
| 0x0010f510 | 8 | 13 | C_I3D_cache_base<9I3D_model> | CreateNew | CreateNew__28C_I3D_cache_base<9I3D_model>CFv |
| 0x0010f520 | 8 | 13 | std::__vector_deleter<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>> | data | data__Q23std104__vector_deleter<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>>Fv |
| 0x0010f530 | 84 | 13 | std::__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0> | {dtor} | __dt__Q23std74__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0>Fv |
| 0x0010f590 | 20 | 13 | std::__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>> | {ctor} | __ct__Q23std76__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>>Fv |
| 0x0010f5b0 | 664 | GLOBAL | C_cache_base_block | Tick | Tick__18C_cache_base_blockFUi |
| 0x0010f850 | 1256 | 13 | C_cache_base_block | ActivateBlock | ActivateBlock__18C_cache_base_blockFUi |
| 0x0010fd40 | 364 | 13 | C_cache_base_block | DeactivateBlock | DeactivateBlock__18C_cache_base_blockFUi |
| 0x0010feb0 | 36 | 13 | C_cache_base_block | IsModelOnlyObject | IsModelOnlyObject__18C_cache_base_blockFUs |
| 0x0010fee0 | 840 | 13 | C_cache_base_block | ShowObject | ShowObject__18C_cache_base_blockFUs |
| 0x00110230 | 636 | LOCAL |  | TransparentModel__FP9I3D_frameb | TransparentModel__FP9I3D_frameb |
| 0x001104b0 | 44 | 13 | I3D_frame | UpdateWMatrix | UpdateWMatrix__9I3D_frameFv |
| 0x001104e0 | 20 | 13 | S_vector | {ctor} | __ct__8S_vectorFfff |
| 0x00110500 | 48 | 13 | C_cache_base_block | HideObject | HideObject__18C_cache_base_blockFP10I3D_object |
| 0x00110530 | 1228 | 13 | C_cache_base_block | ShowModel | ShowModel__18C_cache_base_blockFUsPUs |
| 0x00110a00 | 144 | 13 | C_cache_base_block | HideModel | HideModel__18C_cache_base_blockFUsUs |
| 0x00110a90 | 80 | GLOBAL | C_cache_base_block | FindModel | FindModel__18C_cache_base_blockFUsUs |
| 0x00110ae0 | 316 | GLOBAL | C_cache_base_block | DestroyFrame | DestroyFrame__18C_cache_base_blockFP18S_cache_model_inbb |
| 0x00110c20 | 848 | GLOBAL | C_cache_base_block | VisibleAllFrames | VisibleAllFrames__18C_cache_base_blockFv |
| 0x00110f70 | 8 | 13 | std::__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>> | data | data__Q23std76__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>>Fv |
| 0x00110f80 | 8 | 13 | std::__vector_deleter<17S_cache_model_set,Q23std30allocator<17S_cache_model_set>> | data | data__Q23std76__vector_deleter<17S_cache_model_set,Q23std30allocator<17S_cache_model_set>>Fv |
| 0x00110f90 | 8 | 13 | std::__vector_deleter<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>> | data | data__Q23std78__vector_deleter<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>>Fv |
| 0x00110fa0 | 8 | 13 | std::__vector_deleter<18S_cache_base_block,Q23std31allocator<18S_cache_base_block>> | data | data__Q23std78__vector_deleter<18S_cache_base_block,Q23std31allocator<18S_cache_base_block>>Fv |
| 0x00110fb0 | 1320 | 13 | C_cache_base_block | IsBlockInRange | IsBlockInRange__18C_cache_base_blockFUi9S_vector2f |
| 0x001114e0 | 276 | GLOBAL | C_cache_base_block | ClearUnusedModels | ClearUnusedModels__18C_cache_base_blockFv |
| 0x00111600 | 132 | WEAK | std::__vector_deleter<17S_cache_model_set,Q23std30allocator<17S_cache_model_set>> | clear | clear__Q23std76__vector_deleter<17S_cache_model_set,Q23std30allocator<17S_cache_model_set>>Fv |
| 0x00111690 | 24 | 13 |  | Max<i>__FRCiRCi | Max<i>__FRCiRCi |
| 0x001116b0 | 568 | 13 | std::__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0> | do_assign<PC17S_cache_model_one> | do_assign<PC17S_cache_model_one>__Q23std74__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0>FPC17S_cache_model_onePC17S_cache_model_oneQ23std20forward_iterator_tag |
| 0x001118f0 | 116 | WEAK | std::__vector_imp<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>,0> | erase | erase__Q23std102__vector_imp<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>,0>FPQ223C_I3D_cache<9I3D_model>5S_itm |
| 0x00111970 | 1052 | WEAK | std::__vector_imp<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>,0> | insert | insert__Q23std102__vector_imp<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>,0>FPQ223C_I3D_cache<9I3D_model>5S_itmUiRCQ223C_I3D_cache<9I3D_model>5S_itm |
| 0x00111d90 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>,Ui,1> | swap | swap__Q310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>,Ui,1>FRQ310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>,Ui,1> |
| 0x00111db0 | 1072 | WEAK | std::__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0> | insert | insert__Q23std76__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0>FP18S_cache_model_inbbUiRC18S_cache_model_inbb |
| 0x001121e0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18S_cache_model_inbb>,Ui,1> | swap | swap__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_cache_model_inbb>,Ui,1>FRQ310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_cache_model_inbb>,Ui,1> |
| 0x00112200 | 112 | 13 | S_cache_model_inbb | operator= | __as__18S_cache_model_inbbFRC18S_cache_model_inbb |
| 0x00112270 | 88 | 13 | S_cache_model_inbb | {ctor} | __ct__18S_cache_model_inbbFRC18S_cache_model_inbb |
| 0x001122d0 | 1052 | WEAK | std::__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0> | insert | insert__Q23std74__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0>FP17S_cache_model_oneUiRC17S_cache_model_one |
| 0x001126f0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std30allocator<17S_cache_model_one>,Ui,1> | swap | swap__Q310Metrowerks7details64compressed_pair_imp<Q23std30allocator<17S_cache_model_one>,Ui,1>FRQ310Metrowerks7details64compressed_pair_imp<Q23std30allocator<17S_cache_model_one>,Ui,1> |
| 0x00112710 | 1344 | WEAK | std::__vector_imp<18S_cache_base_block,Q23std31allocator<18S_cache_base_block>,0> | insert | insert__Q23std76__vector_imp<18S_cache_base_block,Q23std31allocator<18S_cache_base_block>,0>FP18S_cache_base_blockUiRC18S_cache_base_block |
| 0x00112c50 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18S_cache_base_block>,Ui,1> | swap | swap__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_cache_base_block>,Ui,1>FRQ310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_cache_base_block>,Ui,1> |
| 0x00112c70 | 80 | 13 | std::__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0> | operator= | __as__Q23std76__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0>FRCQ23std76__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0> |
| 0x00112cc0 | 8 | 13 | Metrowerks::compile_assert<1> | operator= | __as__Q210Metrowerks17compile_assert<1>FRCQ210Metrowerks17compile_assert<1> |
| 0x00112cd0 | 316 | 13 | S_cache_base_block | operator= | __as__18S_cache_base_blockFRC18S_cache_base_block |
| 0x00112e10 | 444 | 13 | S_cache_base_block | {ctor} | __ct__18S_cache_base_blockFRC18S_cache_base_block |
| 0x00112fd0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18S_cache_model_inbb>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_cache_model_inbb>,Ui,1>FRCQ23std31allocator<18S_cache_model_inbb>Ui |
| 0x00112fe0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18S_cache_model_inbb>,Ui,1> | first | first__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_cache_model_inbb>,Ui,1>CFv |
| 0x00112ff0 | 1628 | WEAK | std::__vector_imp<17S_cache_model_set,Q23std30allocator<17S_cache_model_set>,0> | insert | insert__Q23std74__vector_imp<17S_cache_model_set,Q23std30allocator<17S_cache_model_set>,0>FP17S_cache_model_setUiRC17S_cache_model_set |
| 0x00113650 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std30allocator<17S_cache_model_set>,Ui,1> | swap | swap__Q310Metrowerks7details64compressed_pair_imp<Q23std30allocator<17S_cache_model_set>,Ui,1>FRQ310Metrowerks7details64compressed_pair_imp<Q23std30allocator<17S_cache_model_set>,Ui,1> |
| 0x00113670 | 72 | 13 | std::__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0> | operator= | __as__Q23std74__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0>FRCQ23std74__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0> |
| 0x001136c0 | 20 | 13 | std::__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0> | end | end__Q23std74__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0>CFv |
| 0x001136e0 | 8 | 13 | std::__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0> | begin | begin__Q23std74__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0>CFv |
| 0x001136f0 | 20 | 13 | std::__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>> | {ctor} | __ct__Q23std76__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>>FRCQ23std30allocator<17S_cache_model_one> |
| 0x00113710 | 8 | 13 | std::__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>> | alloc | alloc__Q23std76__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>>CFv |
| 0x00113720 | 104 | WEAK | std::__vector_deleter<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>> | {dtor} | __dt__Q23std78__vector_deleter<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>>Fv |
| 0x00113790 | 72 | WEAK | std::__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>> | clear | clear__Q23std76__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>>Fv |
| 0x001137e0 | 104 | WEAK | std::__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>> | {dtor} | __dt__Q23std76__vector_deleter<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>>Fv |
| 0x00113850 | 164 | WEAK | std::__vector_deleter<18S_cache_base_block,Q23std31allocator<18S_cache_base_block>> | clear | clear__Q23std78__vector_deleter<18S_cache_base_block,Q23std31allocator<18S_cache_base_block>>Fv |
| 0x00113900 | 104 | WEAK | std::__vector_deleter<18S_cache_base_block,Q23std31allocator<18S_cache_base_block>> | {dtor} | __dt__Q23std78__vector_deleter<18S_cache_base_block,Q23std31allocator<18S_cache_base_block>>Fv |
| 0x00113970 | 104 | WEAK | std::__vector_deleter<17S_cache_model_set,Q23std30allocator<17S_cache_model_set>> | {dtor} | __dt__Q23std76__vector_deleter<17S_cache_model_set,Q23std30allocator<17S_cache_model_set>>Fv |
| 0x001139e0 | 268 | 13 | std::__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0> | init<PC17S_cache_model_one> | init<PC17S_cache_model_one>__Q23std74__vector_imp<17S_cache_model_one,Q23std30allocator<17S_cache_model_one>,0>FPC17S_cache_model_onePC17S_cache_model_oneQ23std20forward_iterator_tag |
| 0x00113af0 | 300 | 13 | std::__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0> | init<PC18S_cache_model_inbb> | init<PC18S_cache_model_inbb>__Q23std76__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0>FPC18S_cache_model_inbbPC18S_cache_model_inbbQ23std20forward_iterator_tag |
| 0x00113c20 | 680 | 13 | std::__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0> | do_assign<PC18S_cache_model_inbb> | do_assign<PC18S_cache_model_inbb>__Q23std76__vector_imp<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>,0>FPC18S_cache_model_inbbPC18S_cache_model_inbbQ23std20forward_iterator_tag |
| 0x00113ed0 | 72 | WEAK | std::__vector_pod<Ui,Q23std13allocator<Ui>> | operator= | __as__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FRCQ23std38__vector_pod<Ui,Q23std13allocator<Ui>> |
| 0x00113f20 | 76 | WEAK | std::__vector_pod<Ui,Q23std13allocator<Ui>> | {ctor} | __ct__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FRCQ23std38__vector_pod<Ui,Q23std13allocator<Ui>> |
| 0x00113f70 | 104 | WEAK | std::__vector_deleter<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>> | {dtor} | __dt__Q23std104__vector_deleter<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>>Fv |
| 0x00113fe0 | 80 | WEAK | std::__vector_deleter<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>> | clear | clear__Q23std78__vector_deleter<18S_cache_model_inbb,Q23std31allocator<18S_cache_model_inbb>>Fv |
| 0x00114030 | 72 | WEAK | std::__vector_deleter<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>> | clear | clear__Q23std104__vector_deleter<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>>Fv |
| 0x00114080 | 256 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | init<PCUi> | init<PCUi>__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FPCUiPCUiQ23std20forward_iterator_tag |
| 0x00114180 | 304 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | do_assign<PCUi> | do_assign<PCUi>__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FPCUiPCUiQ23std20forward_iterator_tag |
| 0x001142b0 | 108 | GLOBAL | C_actor | {ctor} | __ct__7C_actorFv |
| 0x00114320 | 172 | GLOBAL | C_actor | {dtor} | __dt__7C_actorFv |
| 0x001143d0 | 8 | GLOBAL | C_actor | DuplicateFrom | DuplicateFrom__7C_actorFP7C_actor |
| 0x001143e0 | 252 | GLOBAL | C_actor | Init | Init__7C_actorFP9I3D_frame |
| 0x001144e0 | 140 | GLOBAL | C_actor | InternalPlacementReset | InternalPlacementReset__7C_actorFv |
| 0x00114570 | 252 | GLOBAL | C_actor | ResetPlacement | ResetPlacement__7C_actorFv |
| 0x00114670 | 964 | GLOBAL | C_actor | GameInit | GameInit__7C_actorFv |
| 0x00114a40 | 276 | 13 | S_vector | Normalize | Normalize__8S_vectorFv |
| 0x00114b60 | 144 | GLOBAL |  | ExistFrameInScene__FP9I3D_frame | ExistFrameInScene__FP9I3D_frame |
| 0x00114bf0 | 204 | 13 | S_hlp$2695C_actor_cpp | cbFindFrame | cbFindFrame__21S_hlp$2695C_actor_cppFP9I3D_frameP9I3D_frameRb |
| 0x00114cc0 | 428 | GLOBAL | C_actor | GameDone | GameDone__7C_actorFv |
| 0x00114e70 | 272 | GLOBAL | C_actor | Tick | Tick__7C_actorFUi |
| 0x00114f80 | 8 | 13 | C_actor | Update | Update__7C_actorFUi |
| 0x00114f90 | 8 | 13 | C_actor | AI | AI__7C_actorFUi |
| 0x00114fa0 | 8 | 13 | C_actor | NetDirect | NetDirect__7C_actorFUi |
| 0x00114fb0 | 188 | GLOBAL | C_actor | Actor_SaveGameGetSize | Actor_SaveGameGetSize__7C_actorFP7C_actor |
| 0x00115070 | 296 | GLOBAL | C_actor | Actor_SaveGameSave | Actor_SaveGameSave__7C_actorFPPvP7C_actor |
| 0x001151a0 | 316 | GLOBAL | C_actor | Actor_SaveGameLoad | Actor_SaveGameLoad__7C_actorFPPv |
| 0x001152e0 | 52 | LOCAL |  | ZatocToNa4__Fi | ZatocToNa4__Fi |
| 0x00115320 | 396 | GLOBAL | C_actor | SaveGameGetSize | SaveGameGetSize__7C_actorFv |
| 0x001154b0 | 928 | GLOBAL | C_actor | SaveGameSave | SaveGameSave__7C_actorFPPv |
| 0x00115850 | 1988 | GLOBAL | C_actor | SaveGameLoad | SaveGameLoad__7C_actorFPPv |
| 0x00116020 | 36 | GLOBAL | C_actor | SaveGameLoadGuid | SaveGameLoadGuid__7C_actorFPPv |
| 0x00116050 | 8 | GLOBAL | C_actor | Death | Death__7C_actorFv |
| 0x00116060 | 8 | GLOBAL | C_actor | Collision | Collision__7C_actorFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00116070 | 8 | GLOBAL | C_actor | Hit | Hit__7C_actorF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00116080 | 52 | GLOBAL | C_actor | SetActState | SetActState__7C_actorFQ27C_actor11E_act_state |
| 0x001160c0 | 44 | GLOBAL | C_actor | Rebirth | Rebirth__7C_actorFv |
| 0x001160f0 | 2040 | GLOBAL | C_actor | IsPossibleCollAll | IsPossibleCollAll__7C_actorFR8S_vectorR8S_vectorfR8S_vectorPP7C_actor |
| 0x001168f0 | 48 | 13 | S_vector | operator* | __ml__8S_vectorCFRCf |
| 0x00116920 | 44 | 13 | S_vector | Dot | Dot__8S_vectorCFRC8S_vector |
| 0x00116950 | 8 | GLOBAL | C_actor | IsPossibleColl | IsPossibleColl__7C_actorFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor |
| 0x00116960 | 20 | 13 | I3D_sound | SetOn | SetOn__9I3D_soundFb |
| 0x00116980 | 8 | 13 | C_I3D_cache_base<9I3D_sound> | CreateNew | CreateNew__28C_I3D_cache_base<9I3D_sound>CFv |
| 0x00116990 | 8 | 13 | std::__vector_deleter<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>> | data | data__Q23std104__vector_deleter<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>>Fv |
| 0x001169a0 | 1316 | GLOBAL | C_actor | GetBoundPoints2D | GetBoundPoints2D__7C_actorFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame |
| 0x00116ed0 | 148 | GLOBAL | C_actor | DebustizeFrame | DebustizeFrame__7C_actorFP9I3D_frame |
| 0x00116f70 | 148 | GLOBAL | C_actor | NoShadowFrame | NoShadowFrame__7C_actorFP9I3D_frame |
| 0x00117010 | 324 | GLOBAL | C_actor | AddActorToGlobal | AddActorToGlobal__7C_actorFP7C_actor |
| 0x00117160 | 260 | 13 | gif::vector<P7C_actor,8> | reserve | reserve__Q23gif19vector<P7C_actor,8>Fi |
| 0x00117270 | 128 | GLOBAL | C_actor | DelActorFromGlobal | DelActorFromGlobal__7C_actorFP7C_actor |
| 0x001172f0 | 92 | GLOBAL | C_actor | GetWorldPos | GetWorldPos__7C_actorFv |
| 0x00117350 | 112 | GLOBAL | C_actor | GetWorldDir | GetWorldDir__7C_actorFv |
| 0x001173c0 | 136 | GLOBAL | C_actor | AddFrameForPossibleColl | AddFrameForPossibleColl__7C_actorFP9I3D_framef |
| 0x00117450 | 140 | GLOBAL | C_actor | DelFrameForPossibleColl | DelFrameForPossibleColl__7C_actorFP9I3D_frame |
| 0x001174e0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std39allocator<Q27C_actor16S_posscoll_frame>,Ui,1> | first | first__Q310Metrowerks7details73compressed_pair_imp<Q23std39allocator<Q27C_actor16S_posscoll_frame>,Ui,1>Fv |
| 0x001174f0 | 72 | GLOBAL |  | rnd__Fi | rnd__Fi |
| 0x00117540 | 60 | GLOBAL |  | rnd__Fv | rnd__Fv |
| 0x00117580 | 172 | GLOBAL |  | AngleTo2__FRC9S_vector2RC9S_vector2 | AngleTo2__FRC9S_vector2RC9S_vector2 |
| 0x00117630 | 380 | GLOBAL |  | RotateDir__FR8S_vectorf | RotateDir__FR8S_vectorf |
| 0x001177b0 | 116 | GLOBAL |  | RotateDirNoNorm__FR8S_vectorf | RotateDirNoNorm__FR8S_vectorf |
| 0x00117830 | 328 | GLOBAL |  | CompletionBoudVolume__FP9I3D_frameR11I3D_bvolume | CompletionBoudVolume__FP9I3D_frameR11I3D_bvolume |
| 0x00117980 | 48 | GLOBAL |  | compare_anim_index__FPCvPCv | compare_anim_index__FPCvPCv |
| 0x001179b0 | 188 | GLOBAL |  | InitAnimIndex__Fv | InitAnimIndex__Fv |
| 0x00117a70 | 8 | GLOBAL |  | DoneAnimIndex__Fv | DoneAnimIndex__Fv |
| 0x00117a80 | 140 | GLOBAL |  | GetAnimIndex__FUi | GetAnimIndex__FUi |
| 0x00117b10 | 96 | GLOBAL |  | UnlinkAllFrameChild__FP9I3D_frame | UnlinkAllFrameChild__FP9I3D_frame |
| 0x00117b70 | 80 | 13 | C_actor | Release | Release__7C_actorFv |
| 0x00117bc0 | 8 | 13 | C_actor | SaveData | SaveData__7C_actorFP7C_chunk |
| 0x00117bd0 | 8 | 13 | C_actor | LoadData | LoadData__7C_actorFP7C_chunk |
| 0x00117be0 | 8 | 13 | C_actor | ChangeState | ChangeState__7C_actorFv |
| 0x00117bf0 | 16 | 13 | C_actor | GetActorSpeed | GetActorSpeed__7C_actorFR8S_vector |
| 0x00117c00 | 8 | 13 | C_actor | GetMaterialID | GetMaterialID__7C_actorFP9I3D_frame |
| 0x00117c10 | 8 | 13 | C_actor | GetProgram | GetProgram__7C_actorFv |
| 0x00117c20 | 8 | 13 | C_actor | cbCollTester | cbCollTester__7C_actorFP18tDynamicCollObject |
| 0x00117c30 | 8 | 13 | C_actor | SoundsChange | SoundsChange__7C_actorFi |
| 0x00117c40 | 8 | 13 | C_actor | ChangeSceneInit | ChangeSceneInit__7C_actorFv |
| 0x00117c50 | 8 | 13 | C_actor | ChangeSceneDone | ChangeSceneDone__7C_actorFv |
| 0x00117c60 | 116 | WEAK | std::__vector_imp<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>,0> | erase | erase__Q23std102__vector_imp<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>,0>FPQ223C_I3D_cache<9I3D_sound>5S_itm |
| 0x00117ce0 | 1052 | WEAK | std::__vector_imp<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>,0> | insert | insert__Q23std102__vector_imp<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>,0>FPQ223C_I3D_cache<9I3D_sound>5S_itmUiRCQ223C_I3D_cache<9I3D_sound>5S_itm |
| 0x00118100 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>,Ui,1> | swap | swap__Q310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>,Ui,1>FRQ310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>,Ui,1> |
| 0x00118120 | 1052 | WEAK | std::__vector_imp<Q27C_actor16S_posscoll_frame,Q23std39allocator<Q27C_actor16S_posscoll_frame>,0> | insert | insert__Q23std92__vector_imp<Q27C_actor16S_posscoll_frame,Q23std39allocator<Q27C_actor16S_posscoll_frame>,0>FPQ27C_actor16S_posscoll_frameUiRCQ27C_actor16S_posscoll_frame |
| 0x00118540 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std39allocator<Q27C_actor16S_posscoll_frame>,Ui,1> | swap | swap__Q310Metrowerks7details73compressed_pair_imp<Q23std39allocator<Q27C_actor16S_posscoll_frame>,Ui,1>FRQ310Metrowerks7details73compressed_pair_imp<Q23std39allocator<Q27C_actor16S_posscoll_frame>,Ui,1> |
| 0x00118560 | 1052 | WEAK | std::__vector_imp<9S_vector2,Q23std21allocator<9S_vector2>,0> | insert | insert__Q23std56__vector_imp<9S_vector2,Q23std21allocator<9S_vector2>,0>FP9S_vector2UiRC9S_vector2 |
| 0x00118980 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std21allocator<9S_vector2>,Ui,1> | swap | swap__Q310Metrowerks7details55compressed_pair_imp<Q23std21allocator<9S_vector2>,Ui,1>FRQ310Metrowerks7details55compressed_pair_imp<Q23std21allocator<9S_vector2>,Ui,1> |
| 0x001189a0 | 104 | WEAK | std::__vector_deleter<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>> | {dtor} | __dt__Q23std104__vector_deleter<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>>Fv |
| 0x00118a10 | 104 | WEAK | std::__vector_deleter<Q27C_actor16S_posscoll_frame,Q23std39allocator<Q27C_actor16S_posscoll_frame>> | {dtor} | __dt__Q23std94__vector_deleter<Q27C_actor16S_posscoll_frame,Q23std39allocator<Q27C_actor16S_posscoll_frame>>Fv |
| 0x00118a80 | 104 | WEAK | std::__vector_deleter<9S_vector2,Q23std21allocator<9S_vector2>> | {dtor} | __dt__Q23std58__vector_deleter<9S_vector2,Q23std21allocator<9S_vector2>>Fv |
| 0x00118af0 | 72 | WEAK | std::__vector_deleter<Q27C_actor16S_posscoll_frame,Q23std39allocator<Q27C_actor16S_posscoll_frame>> | clear | clear__Q23std94__vector_deleter<Q27C_actor16S_posscoll_frame,Q23std39allocator<Q27C_actor16S_posscoll_frame>>Fv |
| 0x00118b40 | 72 | WEAK | std::__vector_deleter<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>> | clear | clear__Q23std104__vector_deleter<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>>Fv |
| 0x00118b90 | 72 | WEAK | std::__vector_deleter<9S_vector2,Q23std21allocator<9S_vector2>> | clear | clear__Q23std58__vector_deleter<9S_vector2,Q23std21allocator<9S_vector2>>Fv |
| 0x00118be0 | 92 | 13 | std::vector<Q27C_actor16S_posscoll_frame,Q23std39allocator<Q27C_actor16S_posscoll_frame>> | {dtor} | __dt__Q23std84vector<Q27C_actor16S_posscoll_frame,Q23std39allocator<Q27C_actor16S_posscoll_frame>>Fv |
| 0x00118c40 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std39allocator<Q27C_actor16S_posscoll_frame>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details73compressed_pair_imp<Q23std39allocator<Q27C_actor16S_posscoll_frame>,Ui,1>FUi |
| 0x00118c50 | 104 | 13 | gif::vector<P7C_actor,8> | {dtor} | __dt__Q23gif19vector<P7C_actor,8>Fv |
| 0x00118cc0 | 60 | GLOBAL | C_actor_shot | {ctor} | __ct__12C_actor_shotFv |
| 0x00118d00 | 96 | GLOBAL | C_actor_shot | {dtor} | __dt__12C_actor_shotFv |
| 0x00118d60 | 76 | GLOBAL | C_actor_shot | Init | Init__12C_actor_shotFP9I3D_frame |
| 0x00118db0 | 8 | GLOBAL | C_actor_shot | AI | AI__12C_actor_shotFUi |
| 0x00118dc0 | 680 | GLOBAL | C_actor_shot | Update | Update__12C_actor_shotFUi |
| 0x00119070 | 8 | GLOBAL | C_actor_shot | NetDirect | NetDirect__12C_actor_shotFUi |
| 0x00119080 | 224 | GLOBAL | C_actor_shot | GameInit | GameInit__12C_actor_shotFv |
| 0x00119160 | 88 | GLOBAL | C_actor_shot | GameDone | GameDone__12C_actor_shotFv |
| 0x001191c0 | 8 | GLOBAL | C_actor_shot | SaveGameGetSize | SaveGameGetSize__12C_actor_shotFv |
| 0x001191d0 | 8 | GLOBAL | C_actor_shot | SaveGameSave | SaveGameSave__12C_actor_shotFPPv |
| 0x001191e0 | 8 | GLOBAL | C_actor_shot | SaveGameLoad | SaveGameLoad__12C_actor_shotFPPv |
| 0x001191f0 | 204 | GLOBAL | C_actor_shot | SetWeapon | SetWeapon__12C_actor_shotFi |
| 0x001192c0 | 368 | GLOBAL | C_airplane | {ctor} | __ct__10C_airplaneFv |
| 0x00119430 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std13allocator<Ui>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details47compressed_pair_imp<Q23std13allocator<Ui>,Ui,1>FUi |
| 0x00119440 | 152 | GLOBAL | C_airplane | {dtor} | __dt__10C_airplaneFv |
| 0x001194e0 | 44 | GLOBAL | C_airplane | Init | Init__10C_airplaneFP9I3D_frame |
| 0x00119510 | 688 | GLOBAL | C_airplane | SaveData | SaveData__10C_airplaneFP7C_chunk |
| 0x001197c0 | 1904 | GLOBAL | C_airplane | LoadData | LoadData__10C_airplaneFP7C_chunk |
| 0x00119f30 | 164 | GLOBAL | C_airplane | DuplicateFrom | DuplicateFrom__10C_airplaneFP7C_actor |
| 0x00119fe0 | 2756 | GLOBAL | C_airplane | GameInit | GameInit__10C_airplaneFv |
| 0x0011aab0 | 380 | 13 | I3D_frame | GetDir | GetDir__9I3D_frameFv |
| 0x0011ac30 | 268 | GLOBAL | C_airplane | GameDone | GameDone__10C_airplaneFv |
| 0x0011ad40 | 8 | GLOBAL | C_airplane | Death | Death__10C_airplaneFv |
| 0x0011ad50 | 1316 | GLOBAL | C_airplane | NextNode | NextNode__10C_airplaneFv |
| 0x0011b280 | 56 | 13 | S_vector | operator+ | __pl__8S_vectorCFRC8S_vector |
| 0x0011b2c0 | 36 | 13 | S_quat | {ctor} | __ct__6S_quatFfRC8S_vector |
| 0x0011b2f0 | 44 | 13 | S_vector | Cross | Cross__8S_vectorCFRC8S_vector |
| 0x0011b320 | 404 | 13 | I3D_frame | UpdateMatrix | UpdateMatrix__9I3D_frameFv |
| 0x0011b4c0 | 3396 | GLOBAL | C_airplane | AI_fly | AI_fly__10C_airplaneFUi |
| 0x0011c210 | 44 | 13 | C_web_node | GetPos | GetPos__10C_web_nodeFv |
| 0x0011c240 | 548 | GLOBAL | C_airplane | Vrtule | Vrtule__10C_airplaneFUi |
| 0x0011c470 | 1064 | GLOBAL | C_airplane | AI | AI__10C_airplaneFUi |
| 0x0011c8a0 | 8 | GLOBAL | C_airplane | NetDirect | NetDirect__10C_airplaneFUi |
| 0x0011c8b0 | 2296 | GLOBAL | C_airplane | Update | Update__10C_airplaneFUi |
| 0x0011d1b0 | 544 | GLOBAL | C_airplane | Start | Start__10C_airplaneFUif |
| 0x0011d3d0 | 48 | GLOBAL | C_airplane | Start2 | Start2__10C_airplaneFv |
| 0x0011d400 | 164 | GLOBAL | C_airplane | InitVrtule | InitVrtule__10C_airplaneFv |
| 0x0011d4b0 | 880 | GLOBAL | C_airplane | Hit | Hit__10C_airplaneF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x0011d820 | 616 | GLOBAL | C_airplane | UpdateParticles | UpdateParticles__10C_airplaneFv |
| 0x0011da90 | 1484 | GLOBAL | C_airplane | GetEnginesPos | GetEnginesPos__10C_airplaneFR8S_vectorR8S_vectorR8S_vector |
| 0x0011e060 | 292 | GLOBAL | C_airplane | InitSounds | InitSounds__10C_airplaneFv |
| 0x0011e190 | 148 | GLOBAL | C_airplane | ReleaseSounds | ReleaseSounds__10C_airplaneFv |
| 0x0011e230 | 68 | GLOBAL | C_airplane | IsDestroyed | IsDestroyed__10C_airplaneFv |
| 0x0011e280 | 32 | GLOBAL | C_airplane | SaveGameGetSize | SaveGameGetSize__10C_airplaneFv |
| 0x0011e2a0 | 332 | GLOBAL | C_airplane | SaveGameSave | SaveGameSave__10C_airplaneFPPv |
| 0x0011e3f0 | 416 | GLOBAL | C_airplane | SaveGameLoad | SaveGameLoad__10C_airplaneFPPv |
| 0x0011e590 | 156 | GLOBAL | C_airplane | PlaySound | PlaySound__10C_airplaneFQ210C_airplane16E_AIRPLANE_SOUND |
| 0x0011e630 | 664 | GLOBAL | C_airplane | TickSounds | TickSounds__10C_airplaneFUi |
| 0x0011e8d0 | 276 | GLOBAL | C_airplane | ChangeState | ChangeState__10C_airplaneFv |
| 0x0011e9f0 | 148 | GLOBAL | C_airplane | ShowDamageIndicator | ShowDamageIndicator__10C_airplaneFb |
| 0x0011ea90 | 8 | 13 | C_airplane | GetMaterialID | GetMaterialID__10C_airplaneFP9I3D_frame |
| 0x0011eaa0 | 16 | GLOBAL | C_animations_machine_de_luxe | {ctor} | __ct__28C_animations_machine_de_luxeFv |
| 0x0011eab0 | 68 | GLOBAL | C_animations_machine_de_luxe | {dtor} | __dt__28C_animations_machine_de_luxeFv |
| 0x0011eb00 | 2208 | GLOBAL | C_animations_machine_de_luxe | LoadTrack | LoadTrack__28C_animations_machine_de_luxeFPc |
| 0x0011f3a0 | 292 | 13 | C_chunk | Close | Close__7C_chunkFv |
| 0x0011f4d0 | 228 | 13 | C_chunk | Descend | Descend__7C_chunkFv |
| 0x0011f5c0 | 124 | 13 | S_anim_offset_track | {ctor} | __ct__19S_anim_offset_trackFRC19S_anim_offset_track |
| 0x0011f640 | 348 | 13 | gif::vector<19S_anim_offset_track,8> | reserve | reserve__Q23gif31vector<19S_anim_offset_track,8>Fi |
| 0x0011f7a0 | 8 | 13 | S_vector | {ctor} | __ct__8S_vectorFv |
| 0x0011f7b0 | 184 | 13 | C_chunk | OpenFile | OpenFile__7C_chunkFPCcb |
| 0x0011f870 | 188 | GLOBAL | C_animations_machine_de_luxe | GetTrack | GetTrack__28C_animations_machine_de_luxeFPc |
| 0x0011f930 | 188 | GLOBAL | C_animations_machine_de_luxe | ClearNoBaseTracks | ClearNoBaseTracks__28C_animations_machine_de_luxeFv |
| 0x0011f9f0 | 292 | GLOBAL | C_animations_machine_de_luxe | ClearTracks | ClearTracks__28C_animations_machine_de_luxeFv |
| 0x0011fb20 | 124 | GLOBAL | C_animations_machine_de_luxe | Init | Init__28C_animations_machine_de_luxeFP7C_actor |
| 0x0011fba0 | 124 | GLOBAL | C_animations_machine_de_luxe | Init | Init__28C_animations_machine_de_luxeFP17C_traffic_element |
| 0x0011fc20 | 100 | GLOBAL | C_animations_machine_de_luxe | Init | Init__28C_animations_machine_de_luxeFP9I3D_model |
| 0x0011fc90 | 80 | GLOBAL | C_animations_machine_de_luxe | Reset | Reset__28C_animations_machine_de_luxeFv |
| 0x0011fce0 | 432 | GLOBAL | C_animations_machine_de_luxe | EqualTime | EqualTime__28C_animations_machine_de_luxeFii |
| 0x0011fe90 | 136 | GLOBAL | C_animations_machine_de_luxe | GetActiveAnimTime | GetActiveAnimTime__28C_animations_machine_de_luxeFRUiRUi |
| 0x0011ff20 | 88 | GLOBAL | C_animations_machine_de_luxe | StopAnims | StopAnims__28C_animations_machine_de_luxeFv |
| 0x0011ff80 | 128 | GLOBAL | C_animations_machine_de_luxe | RecompileMatrix | RecompileMatrix__28C_animations_machine_de_luxeFR8S_matrix |
| 0x00120000 | 2848 | GLOBAL | C_animations_machine_de_luxe | SetAnim | SetAnim__28C_animations_machine_de_luxeFPcbbbffbbP8S_vectorbUi |
| 0x00120b20 | 112 | 13 | C_animations_machine_de_luxe | GetAnimBaseDelta | GetAnimBaseDelta__28C_animations_machine_de_luxeFP17I3D_animation_set |
| 0x00120b90 | 8 | 13 | C_I3D_cache_base<17I3D_animation_set> | CreateNew | CreateNew__37C_I3D_cache_base<17I3D_animation_set>CFv |
| 0x00120ba0 | 8 | 13 | std::__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>> | data | data__Q23std122__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>>Fv |
| 0x00120bb0 | 104 | GLOBAL | C_animations_machine_de_luxe | IsActive | IsActive__28C_animations_machine_de_luxeFv |
| 0x00120c20 | 4552 | GLOBAL | C_animations_machine_de_luxe | Tick | Tick__28C_animations_machine_de_luxeFUi |
| 0x00121df0 | 284 | GLOBAL | C_animations_machine_de_luxe | SetExtendedAnim | SetExtendedAnim__28C_animations_machine_de_luxeFP17I3D_animation_setffb |
| 0x00121f10 | 132 | GLOBAL | C_animations_machine_de_luxe | DelExtendedAnim | DelExtendedAnim__28C_animations_machine_de_luxeFf |
| 0x00121fa0 | 360 | GLOBAL | C_animations_machine_de_luxe | GetPlacementFromTCK | GetPlacementFromTCK__28C_animations_machine_de_luxeFPcR8S_vectorR8S_vector |
| 0x00122110 | 108 | GLOBAL | C_animations_machine_de_luxe | GetEndPointOfTCK | GetEndPointOfTCK__28C_animations_machine_de_luxeFPcR8S_vector |
| 0x00122180 | 160 | GLOBAL | C_animations_machine_de_luxe | GetMixingPointOfTCK | GetMixingPointOfTCK__28C_animations_machine_de_luxeFPcR8S_vector |
| 0x00122220 | 480 | GLOBAL | C_animations_machine_de_luxe | StageBlendingTick | StageBlendingTick__28C_animations_machine_de_luxeFUi |
| 0x00122400 | 116 | WEAK | std::__vector_imp<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>,0> | erase | erase__Q23std120__vector_imp<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>,0>FPQ232C_I3D_cache<17I3D_animation_set>5S_itm |
| 0x00122480 | 1052 | WEAK | std::__vector_imp<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>,0> | insert | insert__Q23std120__vector_imp<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>,0>FPQ232C_I3D_cache<17I3D_animation_set>5S_itmUiRCQ232C_I3D_cache<17I3D_animation_set>5S_itm |
| 0x001228a0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>,Ui,1> | swap | swap__Q310Metrowerks7details87compressed_pair_imp<Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>,Ui,1>FRQ310Metrowerks7details87compressed_pair_imp<Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>,Ui,1> |
| 0x001228c0 | 104 | WEAK | std::__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>> | {dtor} | __dt__Q23std122__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>>Fv |
| 0x00122930 | 72 | WEAK | std::__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>> | clear | clear__Q23std122__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>>Fv |
| 0x00122980 | 104 | 13 | gif::vector<19S_anim_offset_track,8> | {dtor} | __dt__Q23gif31vector<19S_anim_offset_track,8>Fv |
| 0x001229f0 | 8 | GLOBAL | C_blood | {ctor} | __ct__7C_bloodFv |
| 0x00122a00 | 68 | GLOBAL | C_blood | {dtor} | __dt__7C_bloodFv |
| 0x00122a50 | 472 | GLOBAL | C_blood | Tick | Tick__7C_bloodFf |
| 0x00122c30 | 32 | GLOBAL | C_blood | Init | Init__7C_bloodFR8S_vectorR8S_vectorfib |
| 0x00122c50 | 52 | GLOBAL | C_blood_manager | {ctor} | __ct__15C_blood_managerFv |
| 0x00122c90 | 100 | GLOBAL | C_blood_manager | {dtor} | __dt__15C_blood_managerFv |
| 0x00122d00 | 8 | GLOBAL | C_blood_manager | GameInit | GameInit__15C_blood_managerFv |
| 0x00122d10 | 96 | GLOBAL | C_blood_manager | GameDone | GameDone__15C_blood_managerFv |
| 0x00122d70 | 244 | GLOBAL | C_blood_manager | Tick | Tick__15C_blood_managerFUi |
| 0x00122e70 | 196 | GLOBAL | C_blood_manager | AddBlood | AddBlood__15C_blood_managerFR8S_vectorR8S_vectorfib |
| 0x00122f40 | 112 | GLOBAL | C_bottle | {ctor} | __ct__8C_bottleFv |
| 0x00122fb0 | 96 | GLOBAL | C_bottle | {dtor} | __dt__8C_bottleFv |
| 0x00123010 | 84 | GLOBAL | C_bottle | Init | Init__8C_bottleFP9I3D_frame |
| 0x00123070 | 8 | GLOBAL | C_bottle | DuplicateFrom | DuplicateFrom__8C_bottleFP7C_actor |
| 0x00123080 | 3252 | GLOBAL | C_bottle | AI | AI__8C_bottleFUi |
| 0x00123d40 | 272 | GLOBAL | C_bottle | Update | Update__8C_bottleFUi |
| 0x00123e50 | 8 | GLOBAL | C_bottle | NetDirect | NetDirect__8C_bottleFUi |
| 0x00123e60 | 768 | GLOBAL | C_bottle | GameInit | GameInit__8C_bottleFv |
| 0x00124160 | 64 | GLOBAL | C_bottle | GameDone | GameDone__8C_bottleFv |
| 0x001241a0 | 32 | GLOBAL | C_bottle | SaveGameGetSize | SaveGameGetSize__8C_bottleFv |
| 0x001241c0 | 264 | GLOBAL | C_bottle | SaveGameSave | SaveGameSave__8C_bottleFPPv |
| 0x001242d0 | 436 | GLOBAL | C_bottle | SaveGameLoad | SaveGameLoad__8C_bottleFPPv |
| 0x00124490 | 452 | GLOBAL | C_bottle | AbsoluteImpulse | AbsoluteImpulse__8C_bottleFR8S_vector |
| 0x00124660 | 708 | GLOBAL | C_bottle | Collision | Collision__8C_bottleFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00124930 | 756 | GLOBAL | C_bottle | Hit | Hit__8C_bottleF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00124c30 | 8 | 13 | C_bottle | SaveData | SaveData__8C_bottleFP7C_chunk |
| 0x00124c40 | 8 | 13 | C_bottle | LoadData | LoadData__8C_bottleFP7C_chunk |
| 0x00124c50 | 60 | GLOBAL | C_box_new | {ctor} | __ct__9C_box_newFv |
| 0x00124c90 | 96 | GLOBAL | C_box_new | {dtor} | __dt__9C_box_newFv |
| 0x00124cf0 | 376 | GLOBAL | C_box_new | Init | Init__9C_box_newFP9I3D_frame |
| 0x00124e70 | 196 | GLOBAL | C_box_new | GameInit | GameInit__9C_box_newFv |
| 0x00124f40 | 8 | GLOBAL | C_box_new | GameDone | GameDone__9C_box_newFv |
| 0x00124f50 | 2828 | GLOBAL | C_box_new | LoadData | LoadData__9C_box_newFP7C_chunk |
| 0x00125a60 | 452 | GLOBAL | C_box_new | SaveData | SaveData__9C_box_newFP7C_chunk |
| 0x00125c30 | 96 | GLOBAL | C_box_new | DuplicateFrom | DuplicateFrom__9C_box_newFP7C_actor |
| 0x00125c90 | 12 | GLOBAL | C_box_new | Collision | Collision__9C_box_newFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00125ca0 | 12 | GLOBAL | C_box_new | Hit | Hit__9C_box_newF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00125cb0 | 212 | GLOBAL | C_bridge | {ctor} | __ct__8C_bridgeFv |
| 0x00125d90 | 192 | GLOBAL | C_bridge | {dtor} | __dt__8C_bridgeFv |
| 0x00125e50 | 340 | GLOBAL | C_bridge | Init | Init__8C_bridgeFP9I3D_frame |
| 0x00125fb0 | 1720 | GLOBAL | C_bridge | AI | AI__8C_bridgeFUi |
| 0x00126670 | 5964 | GLOBAL | C_bridge | Update | Update__8C_bridgeFUi |
| 0x00127dc0 | 8 | GLOBAL | C_bridge | NetDirect | NetDirect__8C_bridgeFUi |
| 0x00127dd0 | 6680 | GLOBAL | C_bridge | GameInit | GameInit__8C_bridgeFv |
| 0x001297f0 | 364 | 13 | gif::vector<Q28C_bridge11S_BridgeCol,8> | reserve | reserve__Q23gif34vector<Q28C_bridge11S_BridgeCol,8>Fi |
| 0x00129960 | 288 | GLOBAL | C_bridge | GameDone | GameDone__8C_bridgeFv |
| 0x00129a80 | 8 | GLOBAL | C_bridge | SaveData | SaveData__8C_bridgeFP7C_chunk |
| 0x00129a90 | 276 | GLOBAL | C_bridge | LoadData | LoadData__8C_bridgeFP7C_chunk |
| 0x00129bb0 | 8 | GLOBAL | C_bridge | SaveGameGetSize | SaveGameGetSize__8C_bridgeFv |
| 0x00129bc0 | 28 | GLOBAL | C_bridge | SaveGameSave | SaveGameSave__8C_bridgeFPPv |
| 0x00129be0 | 8 | GLOBAL | C_bridge | SaveGameLoad | SaveGameLoad__8C_bridgeFPPv |
| 0x00129bf0 | 48 | GLOBAL | C_bridge | ShutDown | ShutDown__8C_bridgeFb |
| 0x00129c20 | 140 | GLOBAL | C_bridge | Collision | Collision__8C_bridgeFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00129cb0 | 8 | 13 | C_bridge | DuplicateFrom | DuplicateFrom__8C_bridgeFP7C_actor |
| 0x00129cc0 | 8 | 13 | G_ParnikUzel | {ctor} | __ct__12G_ParnikUzelFv |
| 0x00129cd0 | 364 | GLOBAL | C_car | {ctor} | __ct__5C_carFv |
| 0x00129e40 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_CARINIT_CAR>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_CARINIT_CAR>,Ui,1>FUi |
| 0x00129e50 | 228 | GLOBAL | C_car | {dtor} | __dt__5C_carFv |
| 0x00129f40 | 2512 | GLOBAL | C_car | Init | Init__5C_carFP9I3D_frame |
| 0x0012a910 | 304 | 13 | C_car::S_INPOINT | {ctor} | __ct__Q25C_car9S_INPOINTFRCQ25C_car9S_INPOINT |
| 0x0012aa40 | 380 | 13 | gif::vector<Q25C_car9S_INPOINT,8> | reserve | reserve__Q23gif28vector<Q25C_car9S_INPOINT,8>Fi |
| 0x0012abc0 | 140 | GLOBAL | C_car | CloseCar | CloseCar__5C_carFv |
| 0x0012ac50 | 364 | GLOBAL | C_car | GetPlacementFromTCK | GetPlacementFromTCK__5C_carFiiR8S_vectorR8S_vector |
| 0x0012adc0 | 308 | GLOBAL | C_car | GetPlacementFromTCK1 | GetPlacementFromTCK1__5C_carFiR8S_vectorR8S_vectorb |
| 0x0012af00 | 128 | GLOBAL | C_car | GetUSindex | GetUSindex__5C_carFPCQ214C_using_object14S_using_object |
| 0x0012af80 | 788 | GLOBAL | C_car | LoadMaterials | LoadMaterials__5C_carFv |
| 0x0012b2a0 | 92 | 13 | std::vector<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>> | {dtor} | __dt__Q23std60vector<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>>Fv |
| 0x0012b300 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14S_VEH_MATERIAL>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_VEH_MATERIAL>,Ui,1>FUi |
| 0x0012b310 | 336 | GLOBAL | C_car | LoadParticles | LoadParticles__5C_carFv |
| 0x0012b460 | 2288 | GLOBAL | C_car | Load_Particle | Load_Particle__5C_carFR14S_VEH_PARTICLEi |
| 0x0012bd50 | 8 | 13 | I3D_emitor | SetAnimMode | SetAnimMode__10I3D_emitorFi |
| 0x0012bd60 | 120 | 13 | ANIM_TEX_INFO | {dtor} | __dt__13ANIM_TEX_INFOFv |
| 0x0012bde0 | 2204 | GLOBAL | C_car | Change_Particle | Change_Particle__5C_carFii |
| 0x0012c680 | 824 | GLOBAL | C_car | LoadTyres | LoadTyres__5C_carFv |
| 0x0012c9c0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6S_SRFC>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_SRFC>,Ui,1>FUi |
| 0x0012c9d0 | 92 | 13 | std::vector<6S_TYRE,Q23std18allocator<6S_TYRE>> | {dtor} | __dt__Q23std42vector<6S_TYRE,Q23std18allocator<6S_TYRE>>Fv |
| 0x0012ca30 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6S_TYRE>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_TYRE>,Ui,1>FUi |
| 0x0012ca40 | 140 | GLOBAL | C_car | ConvertTyres | ConvertTyres__5C_carFP7S_TSRFCP6S_SRFC |
| 0x0012cad0 | 1420 | GLOBAL | C_car | SetOwner | SetOwner__5C_carFP7C_actorif |
| 0x0012d060 | 464 | GLOBAL | C_car | GetSeatProperty | GetSeatProperty__5C_carFiRbRbRbRb |
| 0x0012d230 | 128 | GLOBAL | C_car | GetOwner | GetOwner__5C_carFi |
| 0x0012d2b0 | 8 | GLOBAL | C_car | SetDriver | SetDriver__5C_carFb |
| 0x0012d2c0 | 8 | GLOBAL | C_car | GetDriver | GetDriver__5C_carFv |
| 0x0012d2d0 | 92 | GLOBAL | C_car | DuplicateFrom | DuplicateFrom__5C_carFP7C_actor |
| 0x0012d330 | 280 | GLOBAL | C_car | SaveData | SaveData__5C_carFP7C_chunk |
| 0x0012d450 | 448 | GLOBAL | C_car | LoadData | LoadData__5C_carFP7C_chunk |
| 0x0012d610 | 10548 | GLOBAL | C_car | LoadLevelTables | LoadLevelTables__5C_carFv |
| 0x0012ff50 | 780 | GLOBAL | C_car | _RepairPosition | _RepairPosition__5C_carFb |
| 0x00130260 | 8 | GLOBAL | C_car | SetTrafficCar | SetTrafficCar__5C_carFb |
| 0x00130270 | 8 | GLOBAL | C_car | GetLevelsCount | GetLevelsCount__5C_carFv |
| 0x00130280 | 8 | GLOBAL | C_car | GetActualLevel | GetActualLevel__5C_carFv |
| 0x00130290 | 308 | GLOBAL | C_car | SetActualLevel | SetActualLevel__5C_carFi |
| 0x001303d0 | 3996 | GLOBAL | C_car | GameInit | GameInit__5C_carFv |
| 0x00131370 | 568 | GLOBAL | C_car | GameDone | GameDone__5C_carFv |
| 0x001315b0 | 1248 | GLOBAL | C_car | AI | AI__5C_carFUi |
| 0x00131a90 | 6320 | GLOBAL | C_car | Update | Update__5C_carFUi |
| 0x00133340 | 16 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | operator[] | __vc__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>CFUi |
| 0x00133350 | 8 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | data | data__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>CFv |
| 0x00133360 | 3848 | GLOBAL | C_car | UpdateShadow | UpdateShadow__5C_carFv |
| 0x00134270 | 4664 | GLOBAL | C_car | CarExplosion | CarExplosion__5C_carFUi |
| 0x001354b0 | 8 | 13 | std::__vector_deleter<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>> | data | data__Q23std68__vector_deleter<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>>Fv |
| 0x001354c0 | 352 | GLOBAL | C_car | DeactivateCar | DeactivateCar__5C_carFUi |
| 0x00135620 | 1880 | GLOBAL | C_car | Reset | Reset__5C_carFfb |
| 0x00135d80 | 1560 | GLOBAL | C_car | ResetCar | ResetCar__5C_carFv |
| 0x001363a0 | 84 | GLOBAL | C_car | CalmVehicle | CalmVehicle__5C_carFUifi |
| 0x00136400 | 912 | GLOBAL | C_car | ChcesMeSejmout | ChcesMeSejmout__5C_carFRC8S_vectorf |
| 0x00136790 | 196 | GLOBAL | C_car | SetShadowsToMe | SetShadowsToMe__5C_carFb |
| 0x00136860 | 600 | GLOBAL | C_car | SetTransparency | SetTransparency__5C_carFf |
| 0x00136ac0 | 220 | GLOBAL | C_car | GetMaterialID | GetMaterialID__5C_carFP9I3D_frame |
| 0x00136ba0 | 5424 | GLOBAL | C_car | Hit | Hit__5C_carF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x001380d0 | 680 | GLOBAL | C_car | DetectWheel | DetectWheel__5C_carFiRC8S_vectorb |
| 0x00138380 | 3200 | GLOBAL | C_car | IsPossibleColl | IsPossibleColl__5C_carFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor |
| 0x00139000 | 492 | GLOBAL | C_car | Collision | Collision__5C_carFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001391f0 | 652 | GLOBAL | C_car | Actor_Col_Info | Actor_Col_Info__5C_carFPCvRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorPfP8S_vectorPfRb |
| 0x00139480 | 780 | GLOBAL | C_car | Call_Collision | Call_Collision__5C_carFPCvPCvRC8S_vectorRC8S_vectorRC8S_vectorf |
| 0x00139790 | 688 | GLOBAL | C_car | Call_Collision1 | Call_Collision1__5C_carFPCvPCvRC8S_vectorRC8S_vectorRC8S_vector |
| 0x00139a40 | 220 | GLOBAL | C_car | Actor_Filter | Actor_Filter__5C_carFPCvPCvRb |
| 0x00139b20 | 2920 | GLOBAL | C_car | Was_Collision | Was_Collision__5C_carFPCvRC8S_vectorRC8S_vectorRC8S_vectorfRC8S_vectorPCvPCvb |
| 0x0013a690 | 44 | GLOBAL | C_car | Deform_Vehicle_Part | Deform_Vehicle_Part__5C_carFPCvP9I3D_frameUiRC8S_vectorRC8S_vectorb |
| 0x0013a6c0 | 64 | GLOBAL | C_car | Vehicle_Repair | Vehicle_Repair__5C_carFPCv |
| 0x0013a700 | 128 | GLOBAL | C_car | Break_Glass | Break_Glass__5C_carFPCviRC8S_vectorPCc |
| 0x0013a780 | 8 | GLOBAL | C_car | Wake_Up | Wake_Up__5C_carFPCv |
| 0x0013a790 | 2952 | GLOBAL | C_car | Collision_Filter_Body | Collision_Filter_Body__5C_carFPCvPCvRC8S_vectorRC8S_vector |
| 0x0013b320 | 72 | GLOBAL | C_car | Collision_Filter_Wheel | Collision_Filter_Wheel__5C_carFPCvPCvi |
| 0x0013b370 | 288 | GLOBAL | C_car | Prepare_DropOut_Wheel | Prepare_DropOut_Wheel__5C_carFPCviRC8S_vectorPC8S_vector |
| 0x0013b490 | 1700 | GLOBAL | C_car | Prepare_DropOut | Prepare_DropOut__5C_carFPCviRC8S_vectorPC8S_vector |
| 0x0013bb40 | 252 | GLOBAL | C_car | Drop_Out | Drop_Out__5C_carFP5C_carP9I3D_framePvii |
| 0x0013bc40 | 3956 | GLOBAL | C_car | WriteSaveStruct | WriteSaveStruct__5C_carFP6STCarsP9S_CARINITb |
| 0x0013cbc0 | 6024 | GLOBAL | C_car | ReadSaveStruct | ReadSaveStruct__5C_carFP6STCarsP9S_CARINITb |
| 0x0013e350 | 208 | GLOBAL | C_car | GetInvModelMatrix | GetInvModelMatrix__5C_carFb |
| 0x0013e420 | 336 | GLOBAL | C_car | ShowCarEnergy | ShowCarEnergy__5C_carFv |
| 0x0013e570 | 112 | GLOBAL | C_car | GetAIPos | GetAIPos__5C_carFb |
| 0x0013e5e0 | 664 | GLOBAL | C_car | SetEnterProtect | SetEnterProtect__5C_carFb |
| 0x0013e880 | 60 | GLOBAL | C_car | SetEnterProtect1 | SetEnterProtect1__5C_carFib |
| 0x0013e8c0 | 132 | GLOBAL | C_car | GetCarWheelColl | GetCarWheelColl__5C_carFPbi |
| 0x0013e950 | 188 | GLOBAL | C_car | SetCarWheelColl | SetCarWheelColl__5C_carFPbi |
| 0x0013ea10 | 588 | GLOBAL | C_car | IsCarUsable | IsCarUsable__5C_carFUi |
| 0x0013ec60 | 180 | GLOBAL | C_car | GetNavigationPoints | GetNavigationPoints__5C_carFfR8S_vectorR8S_vectorR8S_vectorR8S_vector |
| 0x0013ed20 | 448 | GLOBAL | C_car | ChangeState | ChangeState__5C_carFv |
| 0x0013eee0 | 224 | GLOBAL | C_car | ShadowCarOn | ShadowCarOn__5C_carFb |
| 0x0013efc0 | 12 | GLOBAL | C_car | GetActorSpeed | GetActorSpeed__5C_carFR8S_vector |
| 0x0013efd0 | 80 | GLOBAL | C_car | SoundsChange | SoundsChange__5C_carFi |
| 0x0013f020 | 20 | GLOBAL | C_car | SetTimeParking | SetTimeParking__5C_carFv |
| 0x0013f040 | 24 | GLOBAL | C_car | GetTimeParking | GetTimeParking__5C_carFv |
| 0x0013f060 | 6180 | GLOBAL | C_car | PrepareCarToShow | PrepareCarToShow__5C_carFP9I3D_model |
| 0x00140890 | 16 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | operator[] | __vc__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FUi |
| 0x001408a0 | 8 | 13 | std::__vector_deleter<6S_AXLE,Q23std18allocator<6S_AXLE>> | data | data__Q23std52__vector_deleter<6S_AXLE,Q23std18allocator<6S_AXLE>>Fv |
| 0x001408b0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6S_AXLE>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_AXLE>,Ui,1>FUi |
| 0x001408c0 | 124 | GLOBAL | C_car | IsSeatLeft | IsSeatLeft__5C_carFi |
| 0x00140940 | 244 | GLOBAL | C_car | OdjebajTuStrechu | OdjebajTuStrechu__5C_carFP9I3D_modelb |
| 0x00140a40 | 100 | GLOBAL | C_car | RefreshActivePlayer | RefreshActivePlayer__5C_carFv |
| 0x00140ab0 | 48 | GLOBAL | C_car | DuplicateDamageAndDeform | DuplicateDamageAndDeform__5C_carFP5C_car |
| 0x00140ae0 | 56 | GLOBAL | C_car | SaveGameGetSize | SaveGameGetSize__5C_carFv |
| 0x00140b20 | 540 | GLOBAL | C_car | SaveGameSave | SaveGameSave__5C_carFPPv |
| 0x00140d40 | 760 | GLOBAL | C_car | SaveGameLoad | SaveGameLoad__5C_carFPPv |
| 0x00141040 | 32 | GLOBAL | C_car | DamageDeformGetSize | DamageDeformGetSize__5C_carFv |
| 0x00141060 | 236 | GLOBAL | C_car | DamageDeformSave | DamageDeformSave__5C_carFPPv |
| 0x00141150 | 264 | GLOBAL | C_car | DamageDeformLoad | DamageDeformLoad__5C_carFPPv |
| 0x00141260 | 80 | LOCAL |  | CbEnumObj__FP9I3D_frameUi | CbEnumObj__FP9I3D_frameUi |
| 0x001412b0 | 12 | LOCAL |  | CbEnumObj1__FP9I3D_frameUi | CbEnumObj1__FP9I3D_frameUi |
| 0x001412c0 | 332 | LOCAL |  | CbEnumRoof__FP9I3D_frameUi | CbEnumRoof__FP9I3D_frameUi |
| 0x00141410 | 968 | LOCAL |  | CbClearDeform__FP9I3D_frameUi | CbClearDeform__FP9I3D_frameUi |
| 0x001417e0 | 12 | 13 | C_car | ChangeSceneInit | ChangeSceneInit__5C_carFv |
| 0x001417f0 | 64 | 13 | C_car | ChangeSceneDone | ChangeSceneDone__5C_carFv |
| 0x00141830 | 24 | 13 |  | Min<i>__FRCiRCi | Min<i>__FRCiRCi |
| 0x00141850 | 116 | WEAK | std::__vector_pod<Ui,Q23std13allocator<Ui>> | erase | erase__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FPUi |
| 0x001418d0 | 1388 | WEAK | std::__vector_imp<13S_CARINIT_CAR,Q23std26allocator<13S_CARINIT_CAR>,0> | insert | insert__Q23std66__vector_imp<13S_CARINIT_CAR,Q23std26allocator<13S_CARINIT_CAR>,0>FP13S_CARINIT_CARUiRC13S_CARINIT_CAR |
| 0x00141e40 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_CARINIT_CAR>,Ui,1> | swap | swap__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_CARINIT_CAR>,Ui,1>FRQ310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_CARINIT_CAR>,Ui,1> |
| 0x00141e60 | 1588 | WEAK | std::__vector_imp<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>,0> | insert | insert__Q23std68__vector_imp<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>,0>FP14S_VEH_MATERIALUiRC14S_VEH_MATERIAL |
| 0x001424a0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14S_VEH_MATERIAL>,Ui,1> | swap | swap__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_VEH_MATERIAL>,Ui,1>FRQ310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_VEH_MATERIAL>,Ui,1> |
| 0x001424c0 | 112 | 13 | S_VEH_MATERIAL | operator= | __as__14S_VEH_MATERIALFRC14S_VEH_MATERIAL |
| 0x00142530 | 1580 | WEAK | std::__vector_imp<6S_TYRE,Q23std18allocator<6S_TYRE>,0> | insert | insert__Q23std50__vector_imp<6S_TYRE,Q23std18allocator<6S_TYRE>,0>FP6S_TYREUiRC6S_TYRE |
| 0x00142b60 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6S_TYRE>,Ui,1> | swap | swap__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_TYRE>,Ui,1>FRQ310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_TYRE>,Ui,1> |
| 0x00142b80 | 1292 | WEAK | std::__vector_imp<6S_SRFC,Q23std18allocator<6S_SRFC>,0> | insert | insert__Q23std50__vector_imp<6S_SRFC,Q23std18allocator<6S_SRFC>,0>FP6S_SRFCUiRC6S_SRFC |
| 0x00143090 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6S_SRFC>,Ui,1> | swap | swap__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_SRFC>,Ui,1>FRQ310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_SRFC>,Ui,1> |
| 0x001430b0 | 112 | 13 | S_SRFC | operator= | __as__6S_SRFCFRC6S_SRFC |
| 0x00143120 | 1332 | WEAK | std::__vector_imp<6S_AXLE,Q23std18allocator<6S_AXLE>,0> | insert | insert__Q23std50__vector_imp<6S_AXLE,Q23std18allocator<6S_AXLE>,0>FP6S_AXLEUiRC6S_AXLE |
| 0x00143660 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6S_AXLE>,Ui,1> | swap | swap__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_AXLE>,Ui,1>FRQ310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_AXLE>,Ui,1> |
| 0x00143680 | 64 | 13 | S_AXLE | operator= | __as__6S_AXLEFRC6S_AXLE |
| 0x001436c0 | 72 | WEAK | std::__vector_deleter<13S_CARINIT_CAR,Q23std26allocator<13S_CARINIT_CAR>> | clear | clear__Q23std68__vector_deleter<13S_CARINIT_CAR,Q23std26allocator<13S_CARINIT_CAR>>Fv |
| 0x00143710 | 104 | WEAK | std::__vector_deleter<13S_CARINIT_CAR,Q23std26allocator<13S_CARINIT_CAR>> | {dtor} | __dt__Q23std68__vector_deleter<13S_CARINIT_CAR,Q23std26allocator<13S_CARINIT_CAR>>Fv |
| 0x00143780 | 80 | WEAK | std::__vector_deleter<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>> | clear | clear__Q23std70__vector_deleter<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>>Fv |
| 0x001437d0 | 104 | WEAK | std::__vector_deleter<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>> | {dtor} | __dt__Q23std70__vector_deleter<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>>Fv |
| 0x00143840 | 80 | WEAK | std::__vector_deleter<6S_TYRE,Q23std18allocator<6S_TYRE>> | clear | clear__Q23std52__vector_deleter<6S_TYRE,Q23std18allocator<6S_TYRE>>Fv |
| 0x00143890 | 104 | WEAK | std::__vector_deleter<6S_TYRE,Q23std18allocator<6S_TYRE>> | {dtor} | __dt__Q23std52__vector_deleter<6S_TYRE,Q23std18allocator<6S_TYRE>>Fv |
| 0x00143900 | 104 | WEAK | std::__vector_deleter<6S_SRFC,Q23std18allocator<6S_SRFC>> | {dtor} | __dt__Q23std52__vector_deleter<6S_SRFC,Q23std18allocator<6S_SRFC>>Fv |
| 0x00143970 | 104 | WEAK | std::__vector_deleter<6S_AXLE,Q23std18allocator<6S_AXLE>> | {dtor} | __dt__Q23std52__vector_deleter<6S_AXLE,Q23std18allocator<6S_AXLE>>Fv |
| 0x001439e0 | 88 | WEAK | std::__vector_deleter<6S_SRFC,Q23std18allocator<6S_SRFC>> | clear | clear__Q23std52__vector_deleter<6S_SRFC,Q23std18allocator<6S_SRFC>>Fv |
| 0x00143a40 | 72 | WEAK | std::__vector_deleter<6S_AXLE,Q23std18allocator<6S_AXLE>> | clear | clear__Q23std52__vector_deleter<6S_AXLE,Q23std18allocator<6S_AXLE>>Fv |
| 0x00143a90 | 160 | GLOBAL | C_car_ex | {ctor} | __ct__8C_car_exFv |
| 0x00143b30 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std34allocator<Q28C_car_ex10S_dyn_coll>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q28C_car_ex10S_dyn_coll>,Ui,1>FUi |
| 0x00143b40 | 252 | GLOBAL | C_car_ex | {dtor} | __dt__8C_car_exFv |
| 0x00143c40 | 32 | GLOBAL | C_car_ex | Init | Init__8C_car_exFP9I3D_frame |
| 0x00143c60 | 2500 | GLOBAL | C_car_ex | GameInit | GameInit__8C_car_exFv |
| 0x00144630 | 5200 | 13 | C_car_ex::S_hlp$2074C_car_ex_cpp | cbEnumProp | cbEnumProp__Q28C_car_ex22S_hlp$2074C_car_ex_cppFP9I3D_frameP8C_car_ex |
| 0x00145a80 | 64 | 13 | C_car_ex::S_clap | {ctor} | __ct__Q28C_car_ex6S_clapFRCQ28C_car_ex6S_clap |
| 0x00145ac0 | 308 | 13 | gif::vector<Q28C_car_ex6S_clap,8> | reserve | reserve__Q23gif28vector<Q28C_car_ex6S_clap,8>Fi |
| 0x00145c00 | 276 | 13 | gif::vector<Q28C_car_ex9S_exh_top,8> | reserve | reserve__Q23gif31vector<Q28C_car_ex9S_exh_top,8>Fi |
| 0x00145d20 | 8 | 13 | std::__vector_deleter<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>> | data | data__Q23std84__vector_deleter<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>>Fv |
| 0x00145d30 | 748 | 13 | gif::vector<Q28C_car_ex6S_clap,8> | push_back | push_back__Q23gif28vector<Q28C_car_ex6S_clap,8>FRCQ28C_car_ex6S_clap |
| 0x00146020 | 732 | 13 | gif::vector<Q28C_car_ex9S_exh_top,8> | push_back | push_back__Q23gif31vector<Q28C_car_ex9S_exh_top,8>FRCQ28C_car_ex9S_exh_top |
| 0x00146300 | 848 | GLOBAL | C_car_ex | GameDone | GameDone__8C_car_exFv |
| 0x00146650 | 32 | GLOBAL | C_car_ex | LoadData | LoadData__8C_car_exFP7C_chunk |
| 0x00146670 | 32 | GLOBAL | C_car_ex | SaveData | SaveData__8C_car_exFP7C_chunk |
| 0x00146690 | 8 | GLOBAL | C_car_ex | DuplicateFrom | DuplicateFrom__8C_car_exFP7C_actor |
| 0x001466a0 | 136 | GLOBAL | C_car_ex | DCAddPerson | DCAddPerson__8C_car_exFP7C_human |
| 0x00146730 | 88 | GLOBAL | C_car_ex | DCDelPerson | DCDelPerson__8C_car_exFP7C_human |
| 0x00146790 | 2980 | GLOBAL | C_car_ex | Update | Update__8C_car_exFUi |
| 0x00147340 | 1180 | WEAK | std::__vector_imp<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>,0> | insert | insert__Q23std82__vector_imp<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>,0>FPQ28C_car_ex10S_dyn_collUiRCQ28C_car_ex10S_dyn_coll |
| 0x001477e0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std34allocator<Q28C_car_ex10S_dyn_coll>,Ui,1> | swap | swap__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q28C_car_ex10S_dyn_coll>,Ui,1>FRQ310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q28C_car_ex10S_dyn_coll>,Ui,1> |
| 0x00147800 | 72 | WEAK | std::__vector_deleter<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>> | clear | clear__Q23std84__vector_deleter<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>>Fv |
| 0x00147850 | 104 | WEAK | std::__vector_deleter<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>> | {dtor} | __dt__Q23std84__vector_deleter<Q28C_car_ex10S_dyn_coll,Q23std34allocator<Q28C_car_ex10S_dyn_coll>>Fv |
| 0x001478c0 | 60 | GLOBAL | C_clock | {ctor} | __ct__7C_clockFv |
| 0x00147900 | 96 | GLOBAL | C_clock | {dtor} | __dt__7C_clockFv |
| 0x00147960 | 192 | GLOBAL | C_clock | Init | Init__7C_clockFP9I3D_frame |
| 0x00147a20 | 8 | GLOBAL | C_clock | DuplicateFrom | DuplicateFrom__7C_clockFP7C_actor |
| 0x00147a30 | 380 | GLOBAL | C_clock | GameInit | GameInit__7C_clockFv |
| 0x00147bb0 | 8 | GLOBAL | C_clock | GameDone | GameDone__7C_clockFv |
| 0x00147bc0 | 516 | GLOBAL | C_clock | AI | AI__7C_clockFUi |
| 0x00147dd0 | 8 | 13 | C_clock | Update | Update__7C_clockFUi |
| 0x00147de0 | 8 | 13 | C_clock | SaveGameGetSize | SaveGameGetSize__7C_clockFv |
| 0x00147df0 | 8 | 13 | C_clock | SaveGameSave | SaveGameSave__7C_clockFPPv |
| 0x00147e00 | 8 | 13 | C_clock | SaveGameLoad | SaveGameLoad__7C_clockFPPv |
| 0x00147e10 | 492 | GLOBAL |  | ModelColiisionsOFF__FP9I3D_frame | ModelColiisionsOFF__FP9I3D_frame |
| 0x00148000 | 964 | 13 | S_hlp$1948C_crash_obj_cpp | CollsOff | CollsOff__25S_hlp$1948C_crash_obj_cppFP9I3D_frame |
| 0x001483d0 | 488 | LOCAL |  | DoChange__FP9I3D_frameRiRb | DoChange__FP9I3D_frameRiRb |
| 0x001485c0 | 2828 | GLOBAL |  | CreateCrashActors__FP9I3D_frameR8S_vectorR8S_vectorfPcbUcibb | CreateCrashActors__FP9I3D_frameR8S_vectorR8S_vectorfPcbUcibb |
| 0x001490d0 | 20 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | end | end__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>Fv |
| 0x001490f0 | 1892 | 13 | S_hlp$2136C_crash_obj_cpp | CreateActor | CreateActor__25S_hlp$2136C_crash_obj_cppFP9I3D_frameP9I3D_modelR8S_vectorP9I3D_framebPcUcfbRQ23std56vector<P11C_crash_obj,Q23std25allocator<P11C_crash_obj>> |
| 0x00149860 | 488 | GLOBAL |  | CreateCrashModel__FP9I3D_frameR8S_vectorR8S_vectorf | CreateCrashModel__FP9I3D_frameR8S_vectorR8S_vectorf |
| 0x00149a50 | 112 | GLOBAL | C_crash_obj | {ctor} | __ct__11C_crash_objFv |
| 0x00149ac0 | 96 | GLOBAL | C_crash_obj | {dtor} | __dt__11C_crash_objFv |
| 0x00149b20 | 940 | GLOBAL | C_crash_obj | AI | AI__11C_crash_objFUi |
| 0x00149ed0 | 8 | GLOBAL | C_crash_obj | Update | Update__11C_crash_objFUi |
| 0x00149ee0 | 8 | GLOBAL | C_crash_obj | NetDirect | NetDirect__11C_crash_objFUi |
| 0x00149ef0 | 8 | GLOBAL | C_crash_obj | SaveGameGetSize | SaveGameGetSize__11C_crash_objFv |
| 0x00149f00 | 32 | GLOBAL | C_crash_obj | SaveGameSave | SaveGameSave__11C_crash_objFPPv |
| 0x00149f20 | 32 | GLOBAL | C_crash_obj | SaveGameLoad | SaveGameLoad__11C_crash_objFPPv |
| 0x00149f40 | 1032 | GLOBAL | C_crash_obj | Init | Init__11C_crash_objFP9I3D_frame |
| 0x0014a350 | 8 | GLOBAL | C_crash_obj | DuplicateFrom | DuplicateFrom__11C_crash_objFP7C_actor |
| 0x0014a360 | 40 | GLOBAL | C_crash_obj | GameInit | GameInit__11C_crash_objFv |
| 0x0014a390 | 116 | GLOBAL | C_crash_obj | GameDone | GameDone__11C_crash_objFv |
| 0x0014a410 | 348 | GLOBAL | C_crash_obj | Collision | Collision__11C_crash_objFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x0014a570 | 200 | GLOBAL | C_crash_obj | Hit | Hit__11C_crash_objF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x0014a640 | 120 | GLOBAL | C_crash_obj | SetModelFileName | SetModelFileName__11C_crash_objFPc |
| 0x0014a6c0 | 388 | GLOBAL | C_crash_obj | DestroyOuterMostCrashObject | DestroyOuterMostCrashObject__11C_crash_objFv |
| 0x0014a850 | 8 | 13 | C_crash_obj | SaveData | SaveData__11C_crash_objFP7C_chunk |
| 0x0014a860 | 8 | 13 | C_crash_obj | LoadData | LoadData__11C_crash_objFP7C_chunk |
| 0x0014a870 | 8 | 13 | C_crash_obj | GetMaterialID | GetMaterialID__11C_crash_objFP9I3D_frame |
| 0x0014a880 | 168 | GLOBAL | C_detector | {ctor} | __ct__10C_detectorFv |
| 0x0014a930 | 124 | GLOBAL | C_detector | {dtor} | __dt__10C_detectorFv |
| 0x0014a9b0 | 64 | GLOBAL | C_detector | Init | Init__10C_detectorFP9I3D_frame |
| 0x0014a9f0 | 80 | GLOBAL | C_detector | DuplicateFrom | DuplicateFrom__10C_detectorFP7C_actor |
| 0x0014aa40 | 104 | GLOBAL | C_detector | GameInit | GameInit__10C_detectorFv |
| 0x0014aab0 | 76 | GLOBAL | C_detector | GameDone | GameDone__10C_detectorFv |
| 0x0014ab00 | 136 | GLOBAL | C_detector | SaveData | SaveData__10C_detectorFP7C_chunk |
| 0x0014ab90 | 288 | GLOBAL | C_detector | LoadData | LoadData__10C_detectorFP7C_chunk |
| 0x0014acb0 | 56 | GLOBAL | C_detector | SaveGameGetSize | SaveGameGetSize__10C_detectorFv |
| 0x0014acf0 | 152 | GLOBAL | C_detector | SaveGameSave | SaveGameSave__10C_detectorFPPv |
| 0x0014ad90 | 284 | GLOBAL | C_detector | SaveGameLoad | SaveGameLoad__10C_detectorFPPv |
| 0x0014aeb0 | 8 | GLOBAL | C_detector | NetDirect | NetDirect__10C_detectorFUi |
| 0x0014aec0 | 64 | GLOBAL | C_detector | AI | AI__10C_detectorFUi |
| 0x0014af00 | 964 | GLOBAL | C_detector | Update | Update__10C_detectorFUi |
| 0x0014b2d0 | 156 | GLOBAL | C_detector | Hit | Hit__10C_detectorF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x0014b370 | 1644 | GLOBAL | C_detector | CreateColls | CreateColls__10C_detectorFb |
| 0x0014b9e0 | 1360 | 13 | C_detector::S_hlp$2205C_detector_cpp | cbDummyCreate | cbDummyCreate__Q210C_detector24S_hlp$2205C_detector_cppFP9I3D_frameUi |
| 0x0014bf30 | 148 | 13 | C_detector::S_hlp$2205C_detector_cpp | cbDummyCount | cbDummyCount__Q210C_detector24S_hlp$2205C_detector_cppFP9I3D_frameUi |
| 0x0014bfd0 | 204 | GLOBAL | C_detector | EraseColls | EraseColls__10C_detectorFv |
| 0x0014c0a0 | 244 | GLOBAL | C_detector | CreateUO | CreateUO__10C_detectorFib |
| 0x0014c1a0 | 124 | GLOBAL | C_detector | UseDetector | UseDetector__10C_detectorFv |
| 0x0014c220 | 8 | GLOBAL | C_detector | DecodeParams | DecodeParams__10C_detectorFPcRQ29C_program13S_instruction |
| 0x0014c230 | 8 | GLOBAL | C_detector | DecodeInstruction | DecodeInstruction__10C_detectorFPcRi |
| 0x0014c240 | 8 | GLOBAL | C_detector | DecompileCommand | DecompileCommand__10C_detectorFiPc |
| 0x0014c250 | 872 | GLOBAL | C_detector | Process | Process__10C_detectorFUi |
| 0x0014c5c0 | 136 | GLOBAL | C_detector | ChangeSceneInit | ChangeSceneInit__10C_detectorFv |
| 0x0014c650 | 156 | GLOBAL | C_detector | ChangeSceneDone | ChangeSceneDone__10C_detectorFv |
| 0x0014c6f0 | 8 | 13 | C_detector | GetProgram | GetProgram__10C_detectorFv |
| 0x0014c700 | 8 | 13 | C_program | CallBack | CallBack__9C_programFQ29C_program11E_cb_action |
| 0x0014c710 | 8 | 13 | C_detector | @128@DecompileCommand | @128@DecompileCommand__10C_detectorFiPc |
| 0x0014c720 | 8 | 13 | C_detector | @128@Process | @128@Process__10C_detectorFUi |
| 0x0014c730 | 8 | 13 | C_detector | @128@DecodeInstruction | @128@DecodeInstruction__10C_detectorFPcRi |
| 0x0014c740 | 8 | 13 | C_detector | @128@DecodeParams | @128@DecodeParams__10C_detectorFPcRQ29C_program13S_instruction |
| 0x0014c750 | 8 | 13 | C_detector | @128@SaveGameLoad | @128@SaveGameLoad__10C_detectorFPPv |
| 0x0014c760 | 8 | 13 | C_detector | @128@SaveGameSave | @128@SaveGameSave__10C_detectorFPPv |
| 0x0014c770 | 8 | 13 | C_detector | @128@SaveGameGetSize | @128@SaveGameGetSize__10C_detectorFv |
| 0x0014c780 | 8 | 13 |  | @128@__dt__10C_detectorFv | @128@__dt__10C_detectorFv |
| 0x0014c790 | 692 | GLOBAL | C_dog | {ctor} | __ct__5C_dogFv |
| 0x0014ca50 | 152 | GLOBAL | C_dog | {dtor} | __dt__5C_dogFv |
| 0x0014caf0 | 44 | GLOBAL | C_dog | Init | Init__5C_dogFP9I3D_frame |
| 0x0014cb20 | 408 | GLOBAL | C_dog | SaveData | SaveData__5C_dogFP7C_chunk |
| 0x0014ccc0 | 1088 | GLOBAL | C_dog | LoadData | LoadData__5C_dogFP7C_chunk |
| 0x0014d100 | 116 | GLOBAL | C_dog | DuplicateFrom | DuplicateFrom__5C_dogFP7C_actor |
| 0x0014d180 | 1784 | GLOBAL | C_dog | GameInit | GameInit__5C_dogFv |
| 0x0014d880 | 56 | GLOBAL | C_dog | LoadTCK | LoadTCK__5C_dogFv |
| 0x0014d8c0 | 564 | GLOBAL | C_dog | GameDone | GameDone__5C_dogFv |
| 0x0014db00 | 1816 | GLOBAL | C_dog | LoseHead | LoseHead__5C_dogFRC8S_vector |
| 0x0014e220 | 1180 | GLOBAL | C_dog | Hit | Hit__5C_dogF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x0014e6c0 | 212 | GLOBAL | C_dog | Death | Death__5C_dogFv |
| 0x0014e7a0 | 2500 | GLOBAL | C_dog | AI | AI__5C_dogFUi |
| 0x0014f170 | 2092 | GLOBAL | C_dog | SolvePath | SolvePath__5C_dogFUi |
| 0x0014f9a0 | 284 | 13 | gif::vector<11blocked_con,2> | reserve | reserve__Q23gif23vector<11blocked_con,2>Fi |
| 0x0014fac0 | 1152 | GLOBAL | C_dog | AI_walk_run | AI_walk_run__5C_dogFUiRC8S_vector |
| 0x0014ff40 | 988 | GLOBAL | C_dog | AI_turn | AI_turn__5C_dogFUiRC8S_vector |
| 0x00150320 | 1492 | GLOBAL | C_dog | ChooseAction | ChooseAction__5C_dogFQ25C_dog7E_stateUc |
| 0x00150900 | 1156 | GLOBAL | C_dog | Update | Update__5C_dogFUi |
| 0x00150d90 | 308 | GLOBAL | C_dog | SetAnim | SetAnim__5C_dogFf |
| 0x00150ed0 | 732 | GLOBAL | C_dog | TestCollision | TestCollision__5C_dogFR8S_vector |
| 0x001511b0 | 1096 | GLOBAL | C_dog | ChooseNextCheckPoint | ChooseNextCheckPoint__5C_dogFb |
| 0x00151600 | 212 | GLOBAL | C_dog | IsBlocked | IsBlocked__5C_dogFUi |
| 0x001516e0 | 308 | GLOBAL | C_dog | ground | ground__5C_dogFUi |
| 0x00151820 | 608 | GLOBAL |  | dog_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi | dog_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi |
| 0x00151a80 | 512 | GLOBAL | C_dog | BloodStep | BloodStep__5C_dogFUi |
| 0x00151c80 | 236 | GLOBAL | C_dog | InitSounds | InitSounds__5C_dogFv |
| 0x00151d70 | 240 | GLOBAL | C_dog | InitSound | InitSound__5C_dogFUi |
| 0x00151e60 | 116 | GLOBAL | C_dog | ReleaseSounds | ReleaseSounds__5C_dogFv |
| 0x00151ee0 | 80 | GLOBAL | C_dog | GetSound | GetSound__5C_dogFUi |
| 0x00151f30 | 68 | GLOBAL | C_dog | StopSound | StopSound__5C_dogFUi |
| 0x00151f80 | 136 | GLOBAL | C_dog | ReturnFromAttackAI | ReturnFromAttackAI__5C_dogFv |
| 0x00152010 | 1012 | GLOBAL | C_dog | TestEnemy | TestEnemy__5C_dogFv |
| 0x00152410 | 444 | GLOBAL | C_dog | UnblockPath | UnblockPath__5C_dogFv |
| 0x001525d0 | 380 | GLOBAL | C_dog | UpdatePath | UpdatePath__5C_dogFRC8S_vector |
| 0x00152750 | 32 | GLOBAL | C_dog | SaveGameGetSize | SaveGameGetSize__5C_dogFv |
| 0x00152770 | 548 | GLOBAL | C_dog | SaveGameSave | SaveGameSave__5C_dogFPPv |
| 0x001529a0 | 1604 | GLOBAL | C_dog | SaveGameLoad | SaveGameLoad__5C_dogFPPv |
| 0x00152ff0 | 216 | GLOBAL | C_dog | Collision | Collision__5C_dogFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001530d0 | 48 | GLOBAL | C_dog | cbCollTester | cbCollTester__5C_dogFP18tDynamicCollObject |
| 0x00153100 | 72 | GLOBAL | C_dog | ChangeSceneInit | ChangeSceneInit__5C_dogFv |
| 0x00153150 | 72 | GLOBAL | C_dog | ChangeSceneDone | ChangeSceneDone__5C_dogFv |
| 0x001531a0 | 312 | GLOBAL | C_door | {ctor} | __ct__6C_doorFv |
| 0x001532e0 | 96 | GLOBAL | C_door | {dtor} | __dt__6C_doorFv |
| 0x00153340 | 32 | GLOBAL | C_door | Init | Init__6C_doorFP9I3D_frame |
| 0x00153360 | 148 | GLOBAL | C_door | DuplicateFrom | DuplicateFrom__6C_doorFP7C_actor |
| 0x00153400 | 416 | GLOBAL | C_door | Hit | Hit__6C_doorF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x001535a0 | 1160 | GLOBAL | C_door | SetState | SetState__6C_doorFQ26C_door7E_stateP7C_actorbb |
| 0x00153a30 | 1648 | GLOBAL | C_door | AI | AI__6C_doorFUi |
| 0x001540a0 | 8 | GLOBAL | C_door | Update | Update__6C_doorFUi |
| 0x001540b0 | 68 | GLOBAL | C_door | NetDirect | NetDirect__6C_doorFUi |
| 0x00154100 | 4192 | GLOBAL | C_door | GameInit | GameInit__6C_doorFv |
| 0x00155160 | 160 | GLOBAL | C_door | GameDone | GameDone__6C_doorFv |
| 0x00155200 | 152 | GLOBAL | C_door | EnableUsingObject | EnableUsingObject__6C_doorFb |
| 0x001552a0 | 572 | GLOBAL | C_door | SaveData | SaveData__6C_doorFP7C_chunk |
| 0x001554e0 | 5044 | GLOBAL | C_door | LoadData | LoadData__6C_doorFP7C_chunk |
| 0x001568a0 | 32 | GLOBAL | C_door | SaveGameGetSize | SaveGameGetSize__6C_doorFv |
| 0x001568c0 | 128 | GLOBAL | C_door | SaveGameSave | SaveGameSave__6C_doorFPPv |
| 0x00156940 | 1000 | GLOBAL | C_door | SaveGameLoad | SaveGameLoad__6C_doorFPPv |
| 0x00156d30 | 8 | GLOBAL | C_door | Collision | Collision__6C_doorFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00156d40 | 712 | GLOBAL | C_door | SetOpenAngle | SetOpenAngle__6C_doorFf |
| 0x00157010 | 168 | GLOBAL | C_door | GetBoundPoints2D | GetBoundPoints2D__6C_doorFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame |
| 0x001570c0 | 72 | GLOBAL | C_door | ChangeState | ChangeState__6C_doorFv |
| 0x00157110 | 80 | GLOBAL | C_door | ChangeSceneInit | ChangeSceneInit__6C_doorFv |
| 0x00157160 | 104 | GLOBAL | C_door | ChangeSceneDone | ChangeSceneDone__6C_doorFv |
| 0x001571d0 | 48 | 13 |  | Max<f>__FRCfRCf | Max<f>__FRCfRCf |
| 0x00157200 | 336 | GLOBAL | C_driver | {ctor} | __ct__8C_driverFv |
| 0x00157350 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std24allocator<11S_LOCKCROSS>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_LOCKCROSS>,Ui,1>FUi |
| 0x00157360 | 260 | GLOBAL | C_driver | {dtor} | __dt__8C_driverFv |
| 0x00157470 | 788 | GLOBAL | C_driver | ComputeRadius | ComputeRadius__8C_driverFii |
| 0x00157790 | 528 | GLOBAL | C_driver | Reset | Reset__8C_driverFv |
| 0x001579a0 | 504 | GLOBAL | C_driver | SetCar | SetCar__8C_driverFP5C_car |
| 0x00157ba0 | 14028 | GLOBAL | C_driver | Tick | Tick__8C_driverFUi |
| 0x0015b270 | 224 | 13 | S_car_node | {ctor} | __ct__10S_car_nodeFRC10S_car_node |
| 0x0015b350 | 348 | 13 | gif::vector<10S_car_node,8> | reserve | reserve__Q23gif22vector<10S_car_node,8>Fi |
| 0x0015b4b0 | 224 | 13 | S_car_node | operator= | __as__10S_car_nodeFRC10S_car_node |
| 0x0015b590 | 828 | GLOBAL | C_driver | FillStartValues | FillStartValues__8C_driverFib |
| 0x0015b8d0 | 852 | GLOBAL | C_driver | OptimizeNextCheckpoints | OptimizeNextCheckpoints__8C_driverFv |
| 0x0015bc30 | 1368 | GLOBAL | C_driver | NevimJakToNazvat | NevimJakToNazvat__8C_driverFv |
| 0x0015c190 | 256 | GLOBAL | C_driver | TakNaCemToUbrzdim | TakNaCemToUbrzdim__8C_driverFv |
| 0x0015c290 | 528 | GLOBAL | C_driver | LastRouteInBrakePathPlusToleranciaLaskaASvoboda | LastRouteInBrakePathPlusToleranciaLaskaASvoboda__8C_driverFf |
| 0x0015c4a0 | 140 | GLOBAL | C_driver | MinSOld | MinSOld__8C_driverFf |
| 0x0015c530 | 356 | GLOBAL | C_driver | LastRouteInBrakePath | LastRouteInBrakePath__8C_driverFv |
| 0x0015c6a0 | 884 | GLOBAL | C_driver | MinV | MinV__8C_driverFv |
| 0x0015ca20 | 3972 | GLOBAL | C_driver | PathCorrection | PathCorrection__8C_driverFv |
| 0x0015d9b0 | 2028 | GLOBAL | C_driver | PrejezdZPruhuDoPruhuAParkovani | PrejezdZPruhuDoPruhuAParkovani__8C_driverFv |
| 0x0015e1a0 | 1468 | GLOBAL | C_driver | PredvidaciKoulePripPrejezduMeziPruhy | PredvidaciKoulePripPrejezduMeziPruhy__8C_driverFv |
| 0x0015e760 | 6068 | GLOBAL | C_driver | PredvidaciKolize | PredvidaciKolize__8C_driverFf |
| 0x0015ff20 | 480 | 13 | g_collision | TestSphereSphere | TestSphereSphere__11g_collisionFRC8S_vectorRC8S_vectorfRC8S_vectorRC8S_vectorf |
| 0x00160100 | 276 | 13 | gif::vector<Q211g_collision22gcol_CurveSphereStruct,8> | reserve | reserve__Q23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>Fi |
| 0x00160220 | 1060 | GLOBAL | C_driver | SnizRychlostVzatacce | SnizRychlostVzatacce__8C_driverFv |
| 0x00160650 | 600 | GLOBAL | C_driver | UtekloPraseZase | UtekloPraseZase__8C_driverFv |
| 0x001608b0 | 1436 | GLOBAL | C_driver | Zatoc | Zatoc__8C_driverFv |
| 0x00160e50 | 796 | GLOBAL | C_driver | DoLights | DoLights__8C_driverFUif |
| 0x00161170 | 232 | GLOBAL | C_driver | DoSounds | DoSounds__8C_driverFUi |
| 0x00161260 | 104 | WEAK | std::__vector_deleter<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>> | {dtor} | __dt__Q23std64__vector_deleter<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>>Fv |
| 0x001612d0 | 88 | WEAK | std::__vector_deleter<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>> | clear | clear__Q23std64__vector_deleter<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>>Fv |
| 0x00161330 | 160 | GLOBAL | C_drop_in_weapon | {ctor} | __ct__16C_drop_in_weaponFv |
| 0x001613d0 | 96 | GLOBAL | C_drop_in_weapon | {dtor} | __dt__16C_drop_in_weaponFv |
| 0x00161430 | 8 | GLOBAL | C_drop_in_weapon | AI | AI__16C_drop_in_weaponFUi |
| 0x00161440 | 104 | GLOBAL | C_drop_in_weapon | Update | Update__16C_drop_in_weaponFUi |
| 0x001614b0 | 8 | GLOBAL | C_drop_in_weapon | NetDirect | NetDirect__16C_drop_in_weaponFUi |
| 0x001614c0 | 588 | GLOBAL | C_drop_in_weapon | ChangeState | ChangeState__16C_drop_in_weaponFv |
| 0x00161710 | 8 | GLOBAL | C_drop_in_weapon | Init | Init__16C_drop_in_weaponFP9I3D_frame |
| 0x00161720 | 8 | GLOBAL | C_drop_in_weapon | DuplicateFrom | DuplicateFrom__16C_drop_in_weaponFP7C_actor |
| 0x00161730 | 228 | GLOBAL | C_drop_in_weapon | GameInit | GameInit__16C_drop_in_weaponFv |
| 0x00161820 | 224 | GLOBAL | C_drop_in_weapon | GameDone | GameDone__16C_drop_in_weaponFv |
| 0x00161900 | 8 | GLOBAL | C_drop_in_weapon | Collision | Collision__16C_drop_in_weaponFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00161910 | 8 | GLOBAL | C_drop_in_weapon | Hit | Hit__16C_drop_in_weaponF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00161920 | 8 | 13 | C_drop_in_weapon | SaveData | SaveData__16C_drop_in_weaponFP7C_chunk |
| 0x00161930 | 8 | 13 | C_drop_in_weapon | LoadData | LoadData__16C_drop_in_weaponFP7C_chunk |
| 0x00161940 | 100 | GLOBAL | C_DropOut | {ctor} | __ct__9C_DropOutFv |
| 0x001619b0 | 148 | GLOBAL | C_DropOut | {dtor} | __dt__9C_DropOutFv |
| 0x00161a50 | 616 | GLOBAL | C_DropOut | Init | Init__9C_DropOutFP9I3D_frame |
| 0x00161cc0 | 48 | GLOBAL | C_DropOut | GameInit | GameInit__9C_DropOutFv |
| 0x00161cf0 | 84 | GLOBAL | C_DropOut | GameDone | GameDone__9C_DropOutFv |
| 0x00161d50 | 76 | GLOBAL | C_DropOut | AI | AI__9C_DropOutFUi |
| 0x00161da0 | 40 | GLOBAL | C_DropOut | Update | Update__9C_DropOutFUi |
| 0x00161dd0 | 172 | GLOBAL | C_DropOut | ChangeState | ChangeState__9C_DropOutFv |
| 0x00161e80 | 8 | GLOBAL | C_DropOut | InitWheel | InitWheel__9C_DropOutFP15S_DROPOUT_WHEEL |
| 0x00161e90 | 56 | GLOBAL | C_DropOut | InitBox | InitBox__9C_DropOutFP13S_DROPOUT_BOX |
| 0x00161ed0 | 44 | GLOBAL | C_DropOut | Collision | Collision__9C_DropOutFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00161f00 | 44 | GLOBAL | C_DropOut | Hit | Hit__9C_DropOutF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00161f30 | 120 | GLOBAL | C_ff_manager | SetVibratoion | SetVibratoion__12C_ff_managerFUsUs |
| 0x00161fb0 | 64 | GLOBAL | C_ff_manager | {ctor} | __ct__12C_ff_managerFv |
| 0x00161ff0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std36allocator<Q212C_ff_manager8S_effect>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details70compressed_pair_imp<Q23std36allocator<Q212C_ff_manager8S_effect>,Ui,1>FUi |
| 0x00162000 | 100 | GLOBAL | C_ff_manager | {dtor} | __dt__12C_ff_managerFv |
| 0x00162070 | 84 | GLOBAL | C_ff_manager | GameInit | GameInit__12C_ff_managerFv |
| 0x001620d0 | 52 | GLOBAL | C_ff_manager | GameDone | GameDone__12C_ff_managerFv |
| 0x00162110 | 1120 | GLOBAL | C_ff_manager | Tick | Tick__12C_ff_managerFUi |
| 0x00162570 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std36allocator<Q212C_ff_manager8S_effect>,Ui,1> | first | first__Q310Metrowerks7details70compressed_pair_imp<Q23std36allocator<Q212C_ff_manager8S_effect>,Ui,1>Fv |
| 0x00162580 | 112 | GLOBAL | C_ff_manager | SetConstVibration | SetConstVibration__12C_ff_managerFiUiUsUs |
| 0x001625f0 | 144 | GLOBAL | C_ff_manager | SetLinearVibration | SetLinearVibration__12C_ff_managerFiUiUsUsUsUs |
| 0x00162680 | 52 | GLOBAL | C_ff_manager | ClearVibrations | ClearVibrations__12C_ff_managerFv |
| 0x001626c0 | 80 | WEAK | std::__vector_deleter<Q212C_ff_manager8S_effect,Q23std36allocator<Q212C_ff_manager8S_effect>> | clear | clear__Q23std88__vector_deleter<Q212C_ff_manager8S_effect,Q23std36allocator<Q212C_ff_manager8S_effect>>Fv |
| 0x00162710 | 1444 | WEAK | std::__vector_imp<Q212C_ff_manager8S_effect,Q23std36allocator<Q212C_ff_manager8S_effect>,0> | insert | insert__Q23std86__vector_imp<Q212C_ff_manager8S_effect,Q23std36allocator<Q212C_ff_manager8S_effect>,0>FPQ212C_ff_manager8S_effectUiRCQ212C_ff_manager8S_effect |
| 0x00162cc0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std36allocator<Q212C_ff_manager8S_effect>,Ui,1> | swap | swap__Q310Metrowerks7details70compressed_pair_imp<Q23std36allocator<Q212C_ff_manager8S_effect>,Ui,1>FRQ310Metrowerks7details70compressed_pair_imp<Q23std36allocator<Q212C_ff_manager8S_effect>,Ui,1> |
| 0x00162ce0 | 88 | 13 | C_ff_manager::S_effect | operator= | __as__Q212C_ff_manager8S_effectFRCQ212C_ff_manager8S_effect |
| 0x00162d40 | 104 | WEAK | std::__vector_deleter<Q212C_ff_manager8S_effect,Q23std36allocator<Q212C_ff_manager8S_effect>> | {dtor} | __dt__Q23std88__vector_deleter<Q212C_ff_manager8S_effect,Q23std36allocator<Q212C_ff_manager8S_effect>>Fv |
| 0x00162db0 | 72 | GLOBAL | C_grenade | {ctor} | __ct__9C_grenadeFv |
| 0x00162e00 | 96 | GLOBAL | C_grenade | {dtor} | __dt__9C_grenadeFv |
| 0x00162e60 | 44 | GLOBAL | C_grenade | Init | Init__9C_grenadeFP9I3D_frame |
| 0x00162e90 | 1756 | GLOBAL | C_grenade | AI | AI__9C_grenadeFUi |
| 0x00163570 | 8 | GLOBAL | C_grenade | Update | Update__9C_grenadeFUi |
| 0x00163580 | 56 | GLOBAL | C_grenade | GameInit | GameInit__9C_grenadeFv |
| 0x001635c0 | 80 | GLOBAL | C_grenade | GameDone | GameDone__9C_grenadeFv |
| 0x00163610 | 32 | GLOBAL | C_grenade | SaveGameGetSize | SaveGameGetSize__9C_grenadeFv |
| 0x00163630 | 168 | GLOBAL | C_grenade | SaveGameSave | SaveGameSave__9C_grenadeFPPv |
| 0x001636e0 | 184 | GLOBAL | C_grenade | SaveGameLoad | SaveGameLoad__9C_grenadeFPPv |
| 0x001637a0 | 76 | GLOBAL | C_grenade | Collision | Collision__9C_grenadeFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001637f0 | 72 | GLOBAL | C_grenade | Hit | Hit__9C_grenadeF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00163840 | 8 | GLOBAL | C_grenade | NetDirect | NetDirect__9C_grenadeFUi |
| 0x00163850 | 8 | 13 | C_grenade | SaveData | SaveData__9C_grenadeFP7C_chunk |
| 0x00163860 | 8 | 13 | C_grenade | LoadData | LoadData__9C_grenadeFP7C_chunk |
| 0x00163870 | 5404 | GLOBAL |  | human_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi | human_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi |
| 0x00164d90 | 88 | GLOBAL | ai_loc | Init | Init__6ai_locFP7C_actor |
| 0x00164df0 | 1136 | GLOBAL | ai_loc | Update | Update__6ai_locFv |
| 0x00165260 | 192 | GLOBAL | ai_loc | AddSegment | AddSegment__6ai_locFUiPv |
| 0x00165320 | 124 | GLOBAL | ai_loc | SegListUpdate | SegListUpdate__6ai_locFv |
| 0x001653a0 | 12 | GLOBAL | ai_blob | operator new | __nw__7ai_blobFUi |
| 0x001653b0 | 16 | GLOBAL | ai_blob | operator delete | __dl__7ai_blobFPv |
| 0x001653c0 | 1128 | GLOBAL | ai_blob_log | Blob | Blob__11ai_blob_logFPv |
| 0x00165830 | 144 | GLOBAL | ai_blob_log | GetBlob | GetBlob__11ai_blob_logFUi |
| 0x001658c0 | 52 | GLOBAL | ai_blob_log | Clear | Clear__11ai_blob_logFv |
| 0x00165900 | 160 | GLOBAL |  | _get_delta_y___FR8S_vector | _get_delta_y___FR8S_vector |
| 0x001659a0 | 160 | GLOBAL |  | _get_delta_nodyn_y___FR8S_vector | _get_delta_nodyn_y___FR8S_vector |
| 0x00165a40 | 644 | GLOBAL | C_human | {ctor} | __ct__7C_humanFv |
| 0x00165cd0 | 20 | 13 | std::__vector_deleter<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>> | {ctor} | __ct__Q23std64__vector_deleter<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>>Fv |
| 0x00165cf0 | 140 | 13 | std::vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>> | {dtor} | __dt__Q23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>Fv |
| 0x00165d80 | 780 | 13 | gif::vector<10S_GameItem,8> | push_back | push_back__Q23gif22vector<10S_GameItem,8>FRC10S_GameItem |
| 0x00166090 | 108 | 13 | gif::vector<10S_GameItem,8> | clear | clear__Q23gif22vector<10S_GameItem,8>Fv |
| 0x00166100 | 104 | 13 | gif::vector<Ui,8> | {dtor} | __dt__Q23gif12vector<Ui,8>Fv |
| 0x00166170 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std24allocator<11S_anim_info>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_anim_info>,Ui,1>FUi |
| 0x00166180 | 380 | GLOBAL | C_human | {dtor} | __dt__7C_humanFv |
| 0x00166300 | 248 | GLOBAL | C_human | Init | Init__7C_humanFP9I3D_frame |
| 0x00166400 | 184 | GLOBAL | C_human | DuplicateFrom | DuplicateFrom__7C_humanFP7C_actor |
| 0x001664c0 | 212 | GLOBAL | C_human | SaveData | SaveData__7C_humanFP7C_chunk |
| 0x001665a0 | 368 | LOCAL |  | _ReadOldCharStruct__FP7C_chunkP7C_human | _ReadOldCharStruct__FP7C_chunkP7C_human |
| 0x00166710 | 292 | LOCAL |  | _ReadOldCharStructVer4__FP7C_chunkP7C_human | _ReadOldCharStructVer4__FP7C_chunkP7C_human |
| 0x00166840 | 1336 | GLOBAL | C_human | LoadData | LoadData__7C_humanFP7C_chunk |
| 0x00166d80 | 92 | GLOBAL | C_human | SaveGameGetSize | SaveGameGetSize__7C_humanFv |
| 0x00166de0 | 920 | GLOBAL | C_human | SaveGameSave | SaveGameSave__7C_humanFPPv |
| 0x00167180 | 2692 | GLOBAL | C_human | SaveGameLoad | SaveGameLoad__7C_humanFPPv |
| 0x00167c10 | 276 | GLOBAL | C_human | Death | Death__7C_humanFv |
| 0x00167d30 | 96 | GLOBAL | C_human | Extern_AI_Update | Extern_AI_Update__7C_humanFUi |
| 0x00167d90 | 208 | GLOBAL | C_human | SoundsChange | SoundsChange__7C_humanFi |
| 0x00167e60 | 6044 | GLOBAL | C_human | AI | AI__7C_humanFUi |
| 0x00169600 | 1480 | GLOBAL | C_human | Intern_UseCar | Intern_UseCar__7C_humanFb |
| 0x00169bd0 | 1720 | GLOBAL | C_human | Intern_UseRailway | Intern_UseRailway__7C_humanFb |
| 0x0016a290 | 1088 | GLOBAL | C_human | _intern_UseCarTruck | _intern_UseCarTruck__7C_humanFP8C_car_exb |
| 0x0016a6d0 | 588 | GLOBAL | C_human | ThrowFromCarTruck | ThrowFromCarTruck__7C_humanFR8S_vectorf |
| 0x0016a920 | 9672 | GLOBAL | C_human | Update | Update__7C_humanFUi |
| 0x0016cef0 | 28 | GLOBAL | C_human | ChangeDeadAnim | ChangeDeadAnim__7C_humanFv |
| 0x0016cf10 | 2616 | GLOBAL | C_human | PersonAnim | PersonAnim__7C_humanFfbb |
| 0x0016d950 | 916 | GLOBAL | C_human | ShiftAnimation | ShiftAnimation__7C_humanFi |
| 0x0016dcf0 | 8 | GLOBAL | C_human | NetDirect | NetDirect__7C_humanFUi |
| 0x0016dd00 | 944 | GLOBAL | C_human | ChangeState | ChangeState__7C_humanFv |
| 0x0016e0b0 | 2540 | GLOBAL | C_human | GameInit | GameInit__7C_humanFv |
| 0x0016eaa0 | 1608 | GLOBAL | C_human | GameDone | GameDone__7C_humanFv |
| 0x0016f0f0 | 844 | GLOBAL | C_human | CreateDynColls | CreateDynColls__7C_humanFv |
| 0x0016f440 | 188 | GLOBAL | C_human | EraseDynColls | EraseDynColls__7C_humanFv |
| 0x0016f500 | 188 | GLOBAL | C_human | DeactivateColls | DeactivateColls__7C_humanFb |
| 0x0016f5c0 | 316 | GLOBAL | C_human | ActivateColls | ActivateColls__7C_humanFi |
| 0x0016f700 | 844 | GLOBAL | C_human | UpdateDynColl | UpdateDynColl__7C_humanFv |
| 0x0016fa50 | 5384 | GLOBAL | C_human | Hit | Hit__7C_humanF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00170f60 | 256 | 13 | C_human | HitLanding | HitLanding__7C_humanFv |
| 0x00171060 | 728 | GLOBAL | C_human | HitInCar | HitInCar__7C_humanF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00171340 | 300 | 13 | C_human | HitAgony | HitAgony__7C_humanFRC8S_vector |
| 0x00171470 | 512 | 13 | C_human | HitStroke | HitStroke__7C_humanFRC8S_vectorUi |
| 0x00171670 | 1144 | 13 | C_human | HitGun | HitGun__7C_humanFRC8S_vectorUib |
| 0x00171af0 | 936 | 13 | C_human | HitGunHPT | HitGunHPT__7C_humanFRC8S_vectorUib |
| 0x00171ea0 | 932 | 13 | C_human | HitFall | HitFall__7C_humanFRC8S_vectorUi |
| 0x00172250 | 1636 | GLOBAL | C_human | Collision | Collision__7C_humanFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001728c0 | 660 | GLOBAL | C_human | IsPossibleColl | IsPossibleColl__7C_humanFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor |
| 0x00172b60 | 2000 | GLOBAL | C_human | PoseInterpolate | PoseInterpolate__7C_humanFUi |
| 0x00173330 | 1816 | GLOBAL | C_human | PoseSetPoseNormal | PoseSetPoseNormal__7C_humanFRC8S_vector |
| 0x00173a50 | 1440 | GLOBAL | C_human | PoseSetPoseAimed | PoseSetPoseAimed__7C_humanFR8S_vector |
| 0x00173ff0 | 84 | GLOBAL | C_human | PoseNullRotate | PoseNullRotate__7C_humanFv |
| 0x00174050 | 104 | GLOBAL | C_human | PoseValidate | PoseValidate__7C_humanFb |
| 0x001740c0 | 368 | GLOBAL | C_human | GetAnimIDForDeath | GetAnimIDForDeath__7C_humanFibRC8S_vectorRC8S_vector |
| 0x00174230 | 5152 | GLOBAL | C_human | Movement | Movement__7C_humanFUi |
| 0x00175650 | 6188 | GLOBAL | C_human | MovementIntern | MovementIntern__7C_humanFUi |
| 0x00176e80 | 300 | 13 | C_human | DynSetNextCollOwnerMult | DynSetNextCollOwnerMult__7C_humanFv |
| 0x00176fb0 | 1288 | GLOBAL | C_human | TestMoveColl | TestMoveColl__7C_humanFRC8S_vectorRC8S_vectorR8S_vector |
| 0x001774c0 | 608 | GLOBAL | C_human | TestMoveCollInOwner | TestMoveCollInOwner__7C_humanFRC8S_vectorRC8S_vectorR8S_vector |
| 0x00177720 | 60 | GLOBAL | C_human | Do_StopAnim | Do_StopAnim__7C_humanFv |
| 0x00177760 | 84 | GLOBAL | C_human | Do_Breath | Do_Breath__7C_humanFv |
| 0x001777c0 | 248 | GLOBAL | C_human | Go_Front | Go_Front__7C_humanFb |
| 0x001778c0 | 292 | GLOBAL | C_human | Go_Back | Go_Back__7C_humanFv |
| 0x001779f0 | 272 | GLOBAL | C_human | Go_Left | Go_Left__7C_humanFb |
| 0x00177b00 | 272 | GLOBAL | C_human | Go_Right | Go_Right__7C_humanFb |
| 0x00177c10 | 592 | GLOBAL | C_human | Go_FrontLeft | Go_FrontLeft__7C_humanFb |
| 0x00177e60 | 592 | GLOBAL | C_human | Go_FrontRight | Go_FrontRight__7C_humanFb |
| 0x001780b0 | 556 | GLOBAL | C_human | Go_BackLeft | Go_BackLeft__7C_humanFv |
| 0x001782e0 | 404 | GLOBAL | C_human | Go_BackRight | Go_BackRight__7C_humanFv |
| 0x00178480 | 436 | GLOBAL | C_human | ChangeAnim | ChangeAnim__7C_humanFPcPc |
| 0x00178640 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std24allocator<11S_anim_info>,Ui,1> | first | first__Q310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_anim_info>,Ui,1>Fv |
| 0x00178650 | 92 | GLOBAL | C_human | GetAnimIDForUseCar | GetAnimIDForUseCar__7C_humanFbbbbbb |
| 0x001786b0 | 1784 | GLOBAL | C_human | Intern_UseCar | Intern_UseCar__7C_humanFP5C_cari |
| 0x00178db0 | 1208 | GLOBAL | C_human | Intern_FromCar | Intern_FromCar__7C_humanFP9I3D_frame |
| 0x00179270 | 984 | GLOBAL | C_human | RepairWeaponPosition | RepairWeaponPosition__7C_humanFv |
| 0x00179650 | 5504 | GLOBAL | C_human | ChangeWeaponModel | ChangeWeaponModel__7C_humanFv |
| 0x0017abd0 | 1596 | GLOBAL | C_human | Do_ChangeWeapon | Do_ChangeWeapon__7C_humanFib |
| 0x0017b210 | 140 | GLOBAL | C_human | Do_Aimed | Do_Aimed__7C_humanFb |
| 0x0017b2a0 | 404 | GLOBAL | C_human | Do_Crouched | Do_Crouched__7C_humanFb |
| 0x0017b440 | 176 | GLOBAL | C_human | InventoryTruncate | InventoryTruncate__7C_humanFv |
| 0x0017b4f0 | 2884 | GLOBAL | C_human | DropOutItems | DropOutItems__7C_humanFRQ23gif22vector<10S_GameItem,8> |
| 0x0017c040 | 468 | GLOBAL | C_human | TozNaselJsemTenNejvyssiMost | TozNaselJsemTenNejvyssiMost__7C_humanFv |
| 0x0017c220 | 916 | GLOBAL | C_human | Can_DropOutFromCar_Free | Can_DropOutFromCar_Free__7C_humanFP5C_cari |
| 0x0017c5c0 | 972 | GLOBAL | C_human | Can_DropOutFromCar | Can_DropOutFromCar__7C_humanFi |
| 0x0017c990 | 1216 | GLOBAL | C_human | Can_EnterToCar | Can_EnterToCar__7C_humanFP5C_carib |
| 0x0017ce50 | 1820 | GLOBAL | C_human | ZasrateLezeniZAuta_ve2rano | ZasrateLezeniZAuta_ve2rano__7C_humanFv |
| 0x0017d570 | 4084 | GLOBAL | C_human | ZasrateVybliknutiZAuta_ve3rano | ZasrateVybliknutiZAuta_ve3rano__7C_humanFv |
| 0x0017e570 | 944 | GLOBAL | C_human | Do_ClimbInCarLR | Do_ClimbInCarLR__7C_humanFv |
| 0x0017e920 | 5420 | GLOBAL | C_human | Use_Actor | Use_Actor__7C_humanFP7C_actoriii |
| 0x0017fe50 | 1168 | GLOBAL | C_human | IsKickToCocotus | IsKickToCocotus__7C_humanFv |
| 0x001802e0 | 12 | 13 | S_matrix | operator() | __cl__8S_matrixFi |
| 0x001802f0 | 3668 | GLOBAL | C_human | Do_Shoot | Do_Shoot__7C_humanFbP8S_vector |
| 0x00181150 | 320 | GLOBAL | C_human | Do_ThrowGrenade | Do_ThrowGrenade__7C_humanFR8S_vector |
| 0x00181290 | 1224 | GLOBAL | C_human | Can_SpecialStroke | Can_SpecialStroke__7C_humanFb |
| 0x00181760 | 324 | GLOBAL | C_human | Do_Castrate | Do_Castrate__7C_humanFP7C_human |
| 0x001818b0 | 324 | GLOBAL | C_human | Do_HardHead | Do_HardHead__7C_humanFP7C_human |
| 0x00181a00 | 108 | 13 | C_human | SetWaitAnim | SetWaitAnim__7C_humanFv |
| 0x00181a70 | 772 | 13 | C_human | SetFreeBoxingAnim | SetFreeBoxingAnim__7C_humanFUi |
| 0x00181d80 | 5024 | GLOBAL | C_human | Shooting | Shooting__7C_humanFv |
| 0x00183120 | 932 | 13 | C_human | FireReload | FireReload__7C_humanFP15S_GameItemTable |
| 0x001834d0 | 472 | GLOBAL | C_human | Do_Reload | Do_Reload__7C_humanFv |
| 0x001836b0 | 388 | GLOBAL | C_human | Do_Holster | Do_Holster__7C_humanFv |
| 0x00183840 | 544 | GLOBAL | C_human | Do_WeaponDrop | Do_WeaponDrop__7C_humanFv |
| 0x00183a60 | 292 | 13 | gif::vector<10S_GameItem,8> | reserve | reserve__Q23gif22vector<10S_GameItem,8>Fi |
| 0x00183b90 | 680 | GLOBAL | C_human | Do_WeaponDropAll | Do_WeaponDropAll__7C_humanFv |
| 0x00183e40 | 252 | GLOBAL | C_human | Do_PlayAnim | Do_PlayAnim__7C_humanFPCcbb |
| 0x00183f40 | 332 | GLOBAL | C_human | GetBoundPoints2D | GetBoundPoints2D__7C_humanFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame |
| 0x00184090 | 716 | GLOBAL | C_human | Do_Jump | Do_Jump__7C_humanFbb |
| 0x00184360 | 3832 | GLOBAL | C_human | Do_Climb | Do_Climb__7C_humanFv |
| 0x00185260 | 448 | GLOBAL | C_human | Do_SideJump | Do_SideJump__7C_humanFRC8S_vectorRC8S_vector |
| 0x00185420 | 3108 | GLOBAL | C_human | _intern_ChangeModel | _intern_ChangeModel__7C_humanFPcb |
| 0x00186050 | 1684 | GLOBAL | C_human | _intern_ForceDeath | _intern_ForceDeath__7C_humanFv |
| 0x001866f0 | 160 | GLOBAL | C_human | Do_ShockStart | Do_ShockStart__7C_humanFi |
| 0x00186790 | 140 | GLOBAL | C_human | Do_ShockDown | Do_ShockDown__7C_humanFi |
| 0x00186820 | 116 | GLOBAL | C_human | Do_ShockUp | Do_ShockUp__7C_humanFv |
| 0x001868a0 | 116 | GLOBAL | C_human | Do_ShockStumble | Do_ShockStumble__7C_humanFv |
| 0x00186920 | 64 | GLOBAL | C_human | cbCollTester | cbCollTester__7C_humanFP18tDynamicCollObject |
| 0x00186960 | 3104 | GLOBAL | C_human | Do_ThrowCocotFromCar | Do_ThrowCocotFromCar__7C_humanFP5C_cari |
| 0x00187580 | 8 | 13 | I3D_frame | Tick | Tick__9I3D_frameFi |
| 0x00187590 | 1496 | GLOBAL | C_human | _intern_ThrowMeFromCar | _intern_ThrowMeFromCar__7C_humanFv |
| 0x00187b70 | 1464 | GLOBAL | C_human | Do_FuckingBox | Do_FuckingBox__7C_humanFP9I3D_frame |
| 0x00188130 | 1772 | GLOBAL | C_human | Do_FuckingBoxCancel | Do_FuckingBoxCancel__7C_humanFv |
| 0x00188820 | 1104 | GLOBAL | C_human | Do_DeadBody | Do_DeadBody__7C_humanFP9I3D_frame |
| 0x00188c70 | 1320 | GLOBAL | C_human | EineMeineKleineAutoInLuft | EineMeineKleineAutoInLuft__7C_humanFP5C_car |
| 0x001891a0 | 296 | GLOBAL | C_human | Can_Aimed | Can_Aimed__7C_humanFi |
| 0x001892d0 | 100 | GLOBAL | C_human | Do_CornerBreath | Do_CornerBreath__7C_humanFQ27C_human13E_corner_view |
| 0x00189340 | 232 | GLOBAL | C_human | Do_CornerView | Do_CornerView__7C_humanFQ27C_human13E_corner_view |
| 0x00189430 | 356 | GLOBAL | C_human | Do_CornerFire | Do_CornerFire__7C_humanFQ27C_human13E_corner_view |
| 0x001895a0 | 296 | GLOBAL | C_human | Do_CornerFireAimed | Do_CornerFireAimed__7C_humanFQ27C_human13E_corner_view |
| 0x001896d0 | 12 | GLOBAL | C_human | Force_CanWorking | Force_CanWorking__7C_humanFv |
| 0x001896e0 | 40 | GLOBAL | C_human | Turn_Turn | Turn_Turn__7C_humanFv |
| 0x00189710 | 48 | GLOBAL | C_human | Turn_Left | Turn_Left__7C_humanFv |
| 0x00189740 | 48 | GLOBAL | C_human | Turn_Right | Turn_Right__7C_humanFv |
| 0x00189770 | 1240 | GLOBAL | C_human | Activate | Activate__7C_humanFv |
| 0x00189c50 | 312 | GLOBAL | C_human | Deactivate | Deactivate__7C_humanFv |
| 0x00189d90 | 1712 | GLOBAL | C_human | NewScream | NewScream__7C_humanFUi |
| 0x0018a440 | 588 | GLOBAL | C_human | Do_FuckingBoxDrop | Do_FuckingBoxDrop__7C_humanFv |
| 0x0018a690 | 156 | GLOBAL | C_human | Do_DeadBodyDrop | Do_DeadBodyDrop__7C_humanFv |
| 0x0018a730 | 136 | GLOBAL | C_human | ResetActions | ResetActions__7C_humanFv |
| 0x0018a7c0 | 8 | GLOBAL | C_human | PickObject | PickObject__7C_humanFR8S_vector |
| 0x0018a7d0 | 8 | GLOBAL | C_human | UniversalUse | UniversalUse__7C_humanFv |
| 0x0018a7e0 | 7528 | GLOBAL | C_human | RecompileDeathPos | RecompileDeathPos__7C_humanFv |
| 0x0018c550 | 284 | GLOBAL | C_human | ProcessDeathPos | ProcessDeathPos__7C_humanFUi |
| 0x0018c670 | 172 | GLOBAL | C_human | IsThisCarStealing | IsThisCarStealing__7C_humanFP5C_car |
| 0x0018c720 | 868 | GLOBAL | C_human | FreeHand | FreeHand__7C_humanFv |
| 0x0018ca90 | 120 | GLOBAL | C_human | BezZasobnikuWeaponEffect | BezZasobnikuWeaponEffect__7C_humanFiP9I3D_model |
| 0x0018cb10 | 2276 | GLOBAL | C_human | StartWeaponEffect | StartWeaponEffect__7C_humanFi |
| 0x0018d400 | 2056 | GLOBAL | C_human | TickWeaponEffect | TickWeaponEffect__7C_humanFUi |
| 0x0018dc10 | 1100 | GLOBAL | C_human | ResetWeaponEffects | ResetWeaponEffects__7C_humanFv |
| 0x0018e060 | 156 | GLOBAL | C_human | FakeWeaponEffect | FakeWeaponEffect__7C_humanFP9I3D_frameib |
| 0x0018e100 | 8 | GLOBAL | C_human | HideLegs | HideLegs__7C_humanFP9I3D_modelb |
| 0x0018e110 | 284 | GLOBAL | C_human | Can_Hit_FilterHumanType | Can_Hit_FilterHumanType__7C_humanFP7C_humanP7C_human |
| 0x0018e230 | 120 | GLOBAL | C_human | ChangeSceneInit | ChangeSceneInit__7C_humanFv |
| 0x0018e2b0 | 160 | GLOBAL | C_human | ChangeSceneDone | ChangeSceneDone__7C_humanFv |
| 0x0018e350 | 180 | GLOBAL | C_human | PlaySoundFromSndTable | PlaySoundFromSndTable__7C_humanFi |
| 0x0018e410 | 980 | GLOBAL | C_human | SchovejPero | SchovejPero__7C_humanFv |
| 0x0018e7f0 | 260 | 13 | C_human::S_hlp$21870C_human_cpp | cbEnumFrm | cbEnumFrm__Q27C_human22S_hlp$21870C_human_cppFP9I3D_frameRQ23std50vector<P9I3D_frame,Q23std22allocator<P9I3D_frame>> |
| 0x0018e900 | 28 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | push_back | push_back__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FRCUi |
| 0x0018e920 | 2568 | GLOBAL | C_human | VystrcPero | VystrcPero__7C_humanFv |
| 0x0018f330 | 12 | 13 | C_human | GetActorSpeed | GetActorSpeed__7C_humanFR8S_vector |
| 0x0018f340 | 72 | WEAK | std::__vector_deleter<11S_anim_info,Q23std24allocator<11S_anim_info>> | clear | clear__Q23std64__vector_deleter<11S_anim_info,Q23std24allocator<11S_anim_info>>Fv |
| 0x0018f390 | 132 | WEAK | std::__vector_imp<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>,0> | erase | erase__Q23std62__vector_imp<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>,0>FP11ai_seg_stat |
| 0x0018f420 | 1180 | WEAK | std::__vector_imp<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>,0> | insert | insert__Q23std62__vector_imp<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>,0>FP11ai_seg_statUiRC11ai_seg_stat |
| 0x0018f8c0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std24allocator<11ai_seg_stat>,Ui,1> | swap | swap__Q310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11ai_seg_stat>,Ui,1>FRQ310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11ai_seg_stat>,Ui,1> |
| 0x0018f8e0 | 1052 | WEAK | std::__vector_imp<11S_anim_info,Q23std24allocator<11S_anim_info>,0> | insert | insert__Q23std62__vector_imp<11S_anim_info,Q23std24allocator<11S_anim_info>,0>FP11S_anim_infoUiRC11S_anim_info |
| 0x0018fd00 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std24allocator<11S_anim_info>,Ui,1> | swap | swap__Q310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_anim_info>,Ui,1>FRQ310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_anim_info>,Ui,1> |
| 0x0018fd20 | 104 | WEAK | std::__vector_deleter<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>> | {dtor} | __dt__Q23std64__vector_deleter<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>>Fv |
| 0x0018fd90 | 104 | WEAK | std::__vector_deleter<11S_anim_info,Q23std24allocator<11S_anim_info>> | {dtor} | __dt__Q23std64__vector_deleter<11S_anim_info,Q23std24allocator<11S_anim_info>>Fv |
| 0x0018fe00 | 72 | WEAK | std::__vector_deleter<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>> | clear | clear__Q23std64__vector_deleter<11ai_seg_stat,Q23std24allocator<11ai_seg_stat>>Fv |
| 0x0018fe50 | 84 | GLOBAL | C_irenka | {ctor} | __ct__8C_irenkaFv |
| 0x0018feb0 | 96 | GLOBAL | C_irenka | {dtor} | __dt__8C_irenkaFv |
| 0x0018ff10 | 44 | GLOBAL | C_irenka | Init | Init__8C_irenkaFP9I3D_frame |
| 0x0018ff40 | 8 | GLOBAL | C_irenka | AI | AI__8C_irenkaFUi |
| 0x0018ff50 | 8 | GLOBAL | C_irenka | Update | Update__8C_irenkaFUi |
| 0x0018ff60 | 8 | GLOBAL | C_irenka | NetDirect | NetDirect__8C_irenkaFUi |
| 0x0018ff70 | 904 | GLOBAL | C_irenka | GameInit | GameInit__8C_irenkaFv |
| 0x00190300 | 108 | GLOBAL | C_irenka | GameDone | GameDone__8C_irenkaFv |
| 0x00190370 | 8 | GLOBAL | C_irenka | SaveGameGetSize | SaveGameGetSize__8C_irenkaFv |
| 0x00190380 | 8 | GLOBAL | C_irenka | SaveGameSave | SaveGameSave__8C_irenkaFPPv |
| 0x00190390 | 8 | GLOBAL | C_irenka | SaveGameLoad | SaveGameLoad__8C_irenkaFPPv |
| 0x001903a0 | 100 | GLOBAL | C_irenka | Deactivate | Deactivate__8C_irenkaFv |
| 0x00190410 | 800 | GLOBAL | C_irenka | ZapichameSi | ZapichameSi__8C_irenkaFRC8S_vectorRC8S_vector |
| 0x00190730 | 1028 | GLOBAL |  | HumanLookTo__FP7C_humanRC8S_vectorUibb | HumanLookTo__FP7C_humanRC8S_vectorUibb |
| 0x00190b40 | 3776 | GLOBAL |  | HumanMoveTo__FP7C_humanR8S_vectorUibUibb | HumanMoveTo__FP7C_humanR8S_vectorUibUibb |
| 0x00191a00 | 44 | GLOBAL |  | FreeTaxiScript__Fv | FreeTaxiScript__Fv |
| 0x00191a30 | 884 | GLOBAL |  | LoadTaxiScript__Fv | LoadTaxiScript__Fv |
| 0x00191db0 | 60 | GLOBAL | C_don_ocas | {ctor} | __ct__10C_don_ocasFv |
| 0x00191df0 | 96 | GLOBAL | C_don_ocas | {dtor} | __dt__10C_don_ocasFv |
| 0x00191e50 | 52 | GLOBAL | C_don_ocas | GameInit | GameInit__10C_don_ocasFv |
| 0x00191e90 | 1264 | GLOBAL | C_don_ocas | IsPossibleColl | IsPossibleColl__10C_don_ocasFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor |
| 0x00192380 | 964 | GLOBAL | C_don_ocas | Collision | Collision__10C_don_ocasFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00192750 | 936 | GLOBAL | C_don_ocas | Hit | Hit__10C_don_ocasF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00192b00 | 332 | GLOBAL | C_don_ocas | GetBoundPoints2D | GetBoundPoints2D__10C_don_ocasFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame |
| 0x00192c50 | 124 | GLOBAL | C_traffic_generator | {ctor} | __ct__19C_traffic_generatorFv |
| 0x00192cd0 | 196 | GLOBAL | C_traffic_generator | {dtor} | __dt__19C_traffic_generatorFv |
| 0x00192da0 | 32 | GLOBAL | C_traffic_generator | Init | Init__19C_traffic_generatorFP9I3D_frame |
| 0x00192dc0 | 388 | GLOBAL | C_traffic_generator | DuplicateFrom | DuplicateFrom__19C_traffic_generatorFP7C_actor |
| 0x00192f50 | 556 | GLOBAL | C_traffic_generator | SaveData | SaveData__19C_traffic_generatorFP7C_chunk |
| 0x00193180 | 2912 | GLOBAL | C_traffic_generator | LoadData | LoadData__19C_traffic_generatorFP7C_chunk |
| 0x00193ce0 | 68 | GLOBAL | C_traffic_generator | SaveGameGetSize | SaveGameGetSize__19C_traffic_generatorFv |
| 0x00193d30 | 944 | GLOBAL | C_traffic_generator | SaveGameSave | SaveGameSave__19C_traffic_generatorFPPv |
| 0x001940e0 | 1268 | GLOBAL | C_traffic_generator | SaveGameLoad | SaveGameLoad__19C_traffic_generatorFPPv |
| 0x001945e0 | 1048 | GLOBAL | C_traffic_generator | GameInit | GameInit__19C_traffic_generatorFv |
| 0x00194a00 | 168 | GLOBAL | C_traffic_generator | GameDone | GameDone__19C_traffic_generatorFv |
| 0x00194ab0 | 272 | GLOBAL |  | DelTaxiPassGlobAct__Fv | DelTaxiPassGlobAct__Fv |
| 0x00194bc0 | 140 | GLOBAL | C_traffic_generator | ChangeSceneInit | ChangeSceneInit__19C_traffic_generatorFv |
| 0x00194c50 | 8 | GLOBAL | C_traffic_generator | ChangeSceneDone | ChangeSceneDone__19C_traffic_generatorFv |
| 0x00194c60 | 48 | GLOBAL | C_traffic_generator | PlayerPromluvilNaTehoChuja | PlayerPromluvilNaTehoChuja__19C_traffic_generatorFP9I3D_frame |
| 0x00194c90 | 364 | GLOBAL | C_traffic_generator | PlayerHoMaVytaseneho | PlayerHoMaVytaseneho__19C_traffic_generatorFv |
| 0x00194e00 | 272 | GLOBAL | C_traffic_generator | SetVisible | SetVisible__19C_traffic_generatorFb |
| 0x00194f10 | 8 | GLOBAL | C_traffic_generator | IsPossibleColl | IsPossibleColl__19C_traffic_generatorFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor |
| 0x00194f20 | 904 | GLOBAL | C_traffic_generator | IsPossibleCollElem | IsPossibleCollElem__19C_traffic_generatorFP17C_traffic_element |
| 0x001952b0 | 8 | GLOBAL | C_traffic_generator | Collision | Collision__19C_traffic_generatorFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001952c0 | 8 | GLOBAL | C_traffic_generator | Hit | Hit__19C_traffic_generatorF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x001952d0 | 728 | GLOBAL | C_traffic_generator | AddElement | AddElement__19C_traffic_generatorFP17C_traffic_element |
| 0x001955b0 | 260 | 13 | gif::vector<P28C_animations_machine_de_luxe,8> | reserve | reserve__Q23gif41vector<P28C_animations_machine_de_luxe,8>Fi |
| 0x001956c0 | 260 | 13 | gif::vector<P17C_traffic_element,8> | reserve | reserve__Q23gif30vector<P17C_traffic_element,8>Fi |
| 0x001957d0 | 240 | GLOBAL | C_traffic_generator | DelElement | DelElement__19C_traffic_generatorFP17C_traffic_element |
| 0x001958c0 | 956 | GLOBAL | C_traffic_generator | AI | AI__19C_traffic_generatorFUi |
| 0x00195c80 | 448 | GLOBAL | C_traffic_generator | TickElements | TickElements__19C_traffic_generatorFUi |
| 0x00195e40 | 148 | GLOBAL | C_traffic_generator | IsCarHasOcas | IsCarHasOcas__19C_traffic_generatorFP5C_car |
| 0x00195ee0 | 2320 | GLOBAL | C_traffic_generator | TryGoToAssHole | TryGoToAssHole__19C_traffic_generatorFv |
| 0x001967f0 | 176 | GLOBAL | C_traffic_generator | PlayerInTaxi | PlayerInTaxi__19C_traffic_generatorFP5C_car |
| 0x001968a0 | 1908 | GLOBAL | C_traffic_generator | GeneratePassengerForTaxi | GeneratePassengerForTaxi__19C_traffic_generatorFv |
| 0x00197020 | 64 | GLOBAL | C_traffic_generator | EnableTaxiDriver | EnableTaxiDriver__19C_traffic_generatorFb |
| 0x00197060 | 140 | GLOBAL | C_traffic_generator | ZrusVsechnyMavajici | ZrusVsechnyMavajici__19C_traffic_generatorFv |
| 0x001970f0 | 324 | GLOBAL | C_traffic_generator | ClearElements | ClearElements__19C_traffic_generatorFv |
| 0x00197240 | 152 | GLOBAL | C_traffic_generator | InvalidateActor | InvalidateActor__19C_traffic_generatorFP7C_actor |
| 0x001972e0 | 1884 | GLOBAL | C_traffic_generator | FizlosNamakatDoAkce | FizlosNamakatDoAkce__19C_traffic_generatorFP7C_humanUiRQ23std46vector<P7C_actor,Q23std20allocator<P7C_actor>>bfb |
| 0x00197a40 | 8 | 13 | std::__vector_deleter<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>> | data | data__Q23std82__vector_deleter<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>>Fv |
| 0x00197a50 | 1128 | GLOBAL | C_traffic_generator | Explosion | Explosion__19C_traffic_generatorFR8S_vectorff |
| 0x00197ec0 | 304 | GLOBAL | C_traffic_generator | Fire | Fire__19C_traffic_generatorFR8S_vectorff |
| 0x00197ff0 | 116 | GLOBAL | C_traffic_generator | GetElemFromFrame | GetElemFromFrame__19C_traffic_generatorFP9I3D_frame |
| 0x00198070 | 140 | GLOBAL | C_traffic_generator | FreeDynCollElements | FreeDynCollElements__19C_traffic_generatorFv |
| 0x00198100 | 232 | GLOBAL | C_traffic_generator | GetNearElementDist2 | GetNearElementDist2__19C_traffic_generatorFR8S_vector |
| 0x001981f0 | 364 | GLOBAL | C_traffic_generator | IsNearElementBefore | IsNearElementBefore__19C_traffic_generatorFP17C_traffic_element |
| 0x00198360 | 8 | GLOBAL | C_traffic_generator | Update | Update__19C_traffic_generatorFUi |
| 0x00198370 | 8 | GLOBAL | C_traffic_generator | NetDirect | NetDirect__19C_traffic_generatorFUi |
| 0x00198380 | 188 | GLOBAL | C_traffic_generator | NewPanic | NewPanic__19C_traffic_generatorFR8S_vector |
| 0x00198440 | 84 | GLOBAL | C_traffic_generator | NewPanic | NewPanic__19C_traffic_generatorFP9I3D_frame |
| 0x001984a0 | 8 | GLOBAL | C_traffic_generator | DetachMesh | DetachMesh__19C_traffic_generatorFP9I3D_model |
| 0x001984b0 | 1344 | GLOBAL | C_traffic_generator | RailwayInStation | RailwayInStation__19C_traffic_generatorFP9C_railwayRQ23std42vector<6rentry,Q23std18allocator<6rentry>> |
| 0x001989f0 | 8 | 13 | std::__vector_deleter<6rentry,Q23std18allocator<6rentry>> | data | data__Q23std52__vector_deleter<6rentry,Q23std18allocator<6rentry>>Fv |
| 0x00198a00 | 116 | GLOBAL | C_traffic_generator | AddFreePerson | AddFreePerson__19C_traffic_generatorFP9I3D_modelP7C_actorb |
| 0x00198a80 | 264 | GLOBAL | C_traffic_generator | AddFreePersonFromCar | AddFreePersonFromCar__19C_traffic_generatorFP9I3D_modelP7C_actorR8S_vectorR8S_vector |
| 0x00198b90 | 1848 | GLOBAL | C_traffic_generator | ElementNewScream | ElementNewScream__19C_traffic_generatorFiiR8S_vectorffff |
| 0x001992d0 | 328 | GLOBAL | C_traffic_generator | VyfunOdTeMasiny | VyfunOdTeMasiny__19C_traffic_generatorFP5C_car |
| 0x00199420 | 152 | GLOBAL | C_traffic_element | {ctor} | __ct__17C_traffic_elementFv |
| 0x001994c0 | 68 | GLOBAL | C_traffic_element | {dtor} | __dt__17C_traffic_elementFv |
| 0x00199510 | 1764 | GLOBAL | C_traffic_element | ObejdiTuPicovinu | ObejdiTuPicovinu__17C_traffic_elementFP7C_actorP9I3D_frame |
| 0x00199c00 | 2456 | GLOBAL | C_traffic_element | SetNewNode | SetNewNode__17C_traffic_elementFv |
| 0x0019a5a0 | 3256 | GLOBAL | C_traffic_element | MavaNaTaxi | MavaNaTaxi__17C_traffic_elementFUi |
| 0x0019b260 | 8 | 13 | std::__vector_pod<Us,Q23std13allocator<Us>> | data | data__Q23std38__vector_pod<Us,Q23std13allocator<Us>>Fv |
| 0x0019b270 | 16 | 13 | std::__vector_imp<Us,Q23std13allocator<Us>,1> | operator[] | __vc__Q23std40__vector_imp<Us,Q23std13allocator<Us>,1>FUi |
| 0x0019b280 | 96 | 13 | std::__vector_pod<Us,Q23std13allocator<Us>> | {dtor} | __dt__Q23std38__vector_pod<Us,Q23std13allocator<Us>>Fv |
| 0x0019b2e0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std13allocator<Us>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details47compressed_pair_imp<Q23std13allocator<Us>,Ui,1>FUi |
| 0x0019b2f0 | 72 | GLOBAL | C_traffic_element | EndMavaniNaTaxi | EndMavaniNaTaxi__17C_traffic_elementFv |
| 0x0019b340 | 16432 | GLOBAL | C_traffic_element | Tick | Tick__17C_traffic_elementFUi |
| 0x0019f370 | 24 | 13 |  | __apl__8S_vectorFRC8S_vector | __apl__8S_vectorFRC8S_vector |
| 0x0019f390 | 444 | 13 | C_traffic_element | UpdateModel | UpdateModel__17C_traffic_elementFv |
| 0x0019f550 | 1076 | GLOBAL | C_traffic_element | SetModel | SetModel__17C_traffic_elementFP9I3D_model |
| 0x0019f990 | 2312 | GLOBAL | C_traffic_element | CreateModel | CreateModel__17C_traffic_elementFPcP19C_traffic_generator |
| 0x001a02a0 | 48 | GLOBAL | C_traffic_element | SetAlpha | SetAlpha__17C_traffic_elementFf |
| 0x001a02d0 | 304 | GLOBAL | C_traffic_element | DestroyModel | DestroyModel__17C_traffic_elementFv |
| 0x001a0400 | 2220 | GLOBAL | C_traffic_element | ActivateFree | ActivateFree__17C_traffic_elementFbb |
| 0x001a0cb0 | 5928 | GLOBAL | C_traffic_element | Activate | Activate__17C_traffic_elementFbff |
| 0x001a23e0 | 16 | 13 | std::__vector_imp<P7C_actor,Q23std20allocator<P7C_actor>,1> | operator[] | __vc__Q23std54__vector_imp<P7C_actor,Q23std20allocator<P7C_actor>,1>FUi |
| 0x001a23f0 | 16 | 13 | S_vector | {ctor} | __ct__8S_vectorFRC8S_vector |
| 0x001a2400 | 432 | GLOBAL | C_traffic_element | Deactivate | Deactivate__17C_traffic_elementFv |
| 0x001a25b0 | 2052 | GLOBAL | C_traffic_element | Hit | Hit__17C_traffic_elementF10E_hit_typeRC8S_vectorUiP7C_actorf |
| 0x001a2dc0 | 628 | GLOBAL | C_traffic_element | SetAnim | SetAnim__17C_traffic_elementFv |
| 0x001a3040 | 760 | 13 | C_traffic_element | HitFall | HitFall__17C_traffic_elementFRC8S_vectorUi |
| 0x001a3340 | 640 | 13 | C_traffic_element | HitGun | HitGun__17C_traffic_elementFRC8S_vectorUib |
| 0x001a35c0 | 476 | 13 | C_traffic_element | HitStroke | HitStroke__17C_traffic_elementFRC8S_vectorUi |
| 0x001a37a0 | 480 | GLOBAL | C_traffic_element | SetPanic | SetPanic__17C_traffic_elementFv |
| 0x001a3980 | 216 | GLOBAL | C_traffic_element | SetMegaPanic | SetMegaPanic__17C_traffic_elementFi |
| 0x001a3a60 | 180 | GLOBAL | C_traffic_element | PlayerNaMneMluvi | PlayerNaMneMluvi__17C_traffic_elementFv |
| 0x001a3b20 | 460 | GLOBAL | C_traffic_element | Karle_hoj | Karle_hoj__17C_traffic_elementFv |
| 0x001a3cf0 | 344 | GLOBAL | C_traffic_element | DropUmbrella | DropUmbrella__17C_traffic_elementFv |
| 0x001a3e50 | 1048 | GLOBAL | C_traffic_element | CreateMafianos | CreateMafianos__17C_traffic_elementF10E_hit_typeRC8S_vectorUiP7C_actorf |
| 0x001a4270 | 140 | 13 | std::vector<P7C_actor,Q23std20allocator<P7C_actor>> | {dtor} | __dt__Q23std46vector<P7C_actor,Q23std20allocator<P7C_actor>>Fv |
| 0x001a4300 | 104 | 13 | std::__vector_imp<P7C_actor,Q23std20allocator<P7C_actor>,1> | {dtor} | __dt__Q23std54__vector_imp<P7C_actor,Q23std20allocator<P7C_actor>,1>Fv |
| 0x001a4370 | 104 | 13 | gif::vector<P28C_animations_machine_de_luxe,8> | {dtor} | __dt__Q23gif41vector<P28C_animations_machine_de_luxe,8>Fv |
| 0x001a43e0 | 104 | 13 | gif::vector<P17C_traffic_element,8> | {dtor} | __dt__Q23gif30vector<P17C_traffic_element,8>Fv |
| 0x001a4450 | 440 | GLOBAL | C_PhThing | {ctor} | __ct__9C_PhThingFv |
| 0x001a4610 | 140 | GLOBAL | C_PhThing | {dtor} | __dt__9C_PhThingFv |
| 0x001a46a0 | 416 | GLOBAL | C_PhThing | Init | Init__9C_PhThingFP9I3D_frame |
| 0x001a4840 | 1328 | GLOBAL | C_PhThing | GameInit | GameInit__9C_PhThingFv |
| 0x001a4d70 | 236 | GLOBAL | C_PhThing | FindActors | FindActors__9C_PhThingFv |
| 0x001a4e60 | 156 | GLOBAL | C_PhThing | GameDone | GameDone__9C_PhThingFv |
| 0x001a4f00 | 504 | GLOBAL | C_PhThing | AI | AI__9C_PhThingFUi |
| 0x001a5100 | 1948 | GLOBAL | C_PhThing | Update | Update__9C_PhThingFUi |
| 0x001a58a0 | 44 | GLOBAL | C_PhThing | ChangeState | ChangeState__9C_PhThingFv |
| 0x001a58d0 | 6276 | GLOBAL | C_PhThing | MoveWheel | MoveWheel__9C_PhThingFf |
| 0x001a7160 | 4456 | GLOBAL | C_PhThing | MoveBox | MoveBox__9C_PhThingFf |
| 0x001a82d0 | 284 | 13 | gif::vector<Q29C_PhThing10S_SURR_ACT,8> | reserve | reserve__Q23gif34vector<Q29C_PhThing10S_SURR_ACT,8>Fi |
| 0x001a83f0 | 1888 | GLOBAL | C_PhThing | MoveCount | MoveCount__9C_PhThingFfRC8S_vector |
| 0x001a8b50 | 1796 | GLOBAL | C_PhThing | CollisionCount | CollisionCount__9C_PhThingFR8S_vectorR8S_vectorR8S_vectorR8S_vectorfUsP7C_actor |
| 0x001a9260 | 332 | GLOBAL | C_PhThing | InitWheel | InitWheel__9C_PhThingFP13S_THING_WHEEL |
| 0x001a93b0 | 1620 | GLOBAL | C_PhThing | InitBox | InitBox__9C_PhThingFP11S_THING_BOX |
| 0x001a9a10 | 532 | GLOBAL | C_PhThing | Collision | Collision__9C_PhThingFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001a9c30 | 1588 | GLOBAL | C_PhThing | Hit | Hit__9C_PhThingF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x001aa270 | 296 | GLOBAL | C_PhThing | GetRVelMassJ | GetRVelMassJ__9C_PhThingFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorPfP8S_vector |
| 0x001aa3a0 | 288 | GLOBAL | C_PhThing | ExternalImpuls | ExternalImpuls__9C_PhThingFRC8S_vectorRC8S_vector |
| 0x001aa4c0 | 216 | GLOBAL | C_PhThing | InvalidateActor | InvalidateActor__9C_PhThingFP7C_actor |
| 0x001aa5a0 | 180 | GLOBAL | C_PhThing | CollisionFilter | CollisionFilter__9C_PhThingFPUiR8S_vectorR8S_vectorRb |
| 0x001aa660 | 440 | GLOBAL | C_PhThing | UpdateImportantVariables | UpdateImportantVariables__9C_PhThingFv |
| 0x001aa820 | 32 | GLOBAL | C_PhThing | SaveGameGetSize | SaveGameGetSize__9C_PhThingFv |
| 0x001aa840 | 564 | GLOBAL | C_PhThing | SaveGameSave | SaveGameSave__9C_PhThingFPPv |
| 0x001aaa80 | 1064 | GLOBAL | C_PhThing | SaveGameLoad | SaveGameLoad__9C_PhThingFPPv |
| 0x001aaeb0 | 8 | GLOBAL | C_PhThing | NetDirect | NetDirect__9C_PhThingFUi |
| 0x001aaec0 | 8 | GLOBAL |  | SetPlayerNaviPoint__FP9I3D_frame | SetPlayerNaviPoint__FP9I3D_frame |
| 0x001aaed0 | 160 | GLOBAL | C_player | {ctor} | __ct__8C_playerFv |
| 0x001aaf70 | 180 | GLOBAL | C_player | {dtor} | __dt__8C_playerFv |
| 0x001ab030 | 60 | GLOBAL | C_player | Init | Init__8C_playerFP9I3D_frame |
| 0x001ab070 | 8 | GLOBAL | C_player | DuplicateFrom | DuplicateFrom__8C_playerFP7C_actor |
| 0x001ab080 | 32 | GLOBAL | C_player | SaveData | SaveData__8C_playerFP7C_chunk |
| 0x001ab0a0 | 32 | GLOBAL | C_player | LoadData | LoadData__8C_playerFP7C_chunk |
| 0x001ab0c0 | 32 | GLOBAL | C_player | SaveGameGetSize | SaveGameGetSize__8C_playerFv |
| 0x001ab0e0 | 176 | GLOBAL | C_player | SaveGameSave | SaveGameSave__8C_playerFPPv |
| 0x001ab190 | 520 | GLOBAL | C_player | SaveGameLoad | SaveGameLoad__8C_playerFPPv |
| 0x001ab3a0 | 8 | GLOBAL | C_player | Death | Death__8C_playerFv |
| 0x001ab3b0 | 1604 | GLOBAL | C_player | FireMagnetism | FireMagnetism__8C_playerFP8S_vectorP8S_vector |
| 0x001aba00 | 932 | GLOBAL | C_player | FireDriveFindTarget | FireDriveFindTarget__8C_playerFP8S_vector |
| 0x001abdb0 | 1548 | GLOBAL | C_player | FindNearestEnemyToTurn | FindNearestEnemyToTurn__8C_playerFb |
| 0x001ac3c0 | 888 | GLOBAL | C_player | ZkurvenaPrebijeciFunkceAdvanced | ZkurvenaPrebijeciFunkceAdvanced__8C_playerFv |
| 0x001ac740 | 10720 | GLOBAL | C_player | AI_walk | AI_walk__8C_playerFUiP17G_ConsoleControls |
| 0x001af120 | 2652 | GLOBAL | C_player | AI_drive | AI_drive__8C_playerFUiP17G_ConsoleControls |
| 0x001afb80 | 2572 | GLOBAL | C_player | AI_car_fire | AI_car_fire__8C_playerFUiP17G_ConsoleControls |
| 0x001b0590 | 3004 | GLOBAL | C_player | AI | AI__8C_playerFUi |
| 0x001b1150 | 1716 | GLOBAL |  | CreateCarMap__FP5C_carR9S_car_mapb | CreateCarMap__FP5C_carR9S_car_mapb |
| 0x001b1810 | 308 | GLOBAL |  | RepairGoToCarPath__FP8C_playerRQ28C_player10S_goto_car | RepairGoToCarPath__FP8C_playerRQ28C_player10S_goto_car |
| 0x001b1950 | 440 | GLOBAL |  | MuzuSeTamNasomrovat__FP7C_humanP5C_cari | MuzuSeTamNasomrovat__FP7C_humanP5C_cari |
| 0x001b1b10 | 4476 | GLOBAL | C_player | GoToCar_Init | GoToCar_Init__8C_playerFv |
| 0x001b2c90 | 260 | 13 | gif::vector<8S_vector,8> | reserve | reserve__Q23gif19vector<8S_vector,8>Fi |
| 0x001b2da0 | 684 | 13 | gif::vector<8S_vector,8> | push_back | push_back__Q23gif19vector<8S_vector,8>FRC8S_vector |
| 0x001b3050 | 108 | 13 | gif::vector<8S_vector,8> | clear | clear__Q23gif19vector<8S_vector,8>Fv |
| 0x001b30c0 | 56 | 13 | I3D_frame | GetWorldPos | GetWorldPos__9I3D_frameFv |
| 0x001b3100 | 348 | 13 | I3D_frame | GetWorldDir | GetWorldDir__9I3D_frameFv |
| 0x001b3260 | 8 | 13 | C_actor | GetFrame | GetFrame__7C_actorFv |
| 0x001b3270 | 24 | GLOBAL | C_player | GoToCar_Cancel | GoToCar_Cancel__8C_playerFv |
| 0x001b3290 | 1612 | GLOBAL | C_player | GoToCar_Process | GoToCar_Process__8C_playerFUiP17G_ConsoleControls |
| 0x001b38e0 | 44 | GLOBAL | C_player | Update | Update__8C_playerFUi |
| 0x001b3910 | 16 | GLOBAL | C_player | NetDirect | NetDirect__8C_playerFUi |
| 0x001b3920 | 320 | GLOBAL | C_player | GameInit | GameInit__8C_playerFv |
| 0x001b3a60 | 92 | GLOBAL | C_player | GameDone | GameDone__8C_playerFv |
| 0x001b3ac0 | 176 | GLOBAL | C_player | Collision | Collision__8C_playerFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001b3b70 | 396 | GLOBAL | C_player | Hit | Hit__8C_playerF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x001b3d00 | 96 | GLOBAL | C_player | UpdateControls | UpdateControls__8C_playerFv |
| 0x001b3d60 | 3288 | GLOBAL | C_player | Do_AB_OwnerNULL | Do_AB_OwnerNULL__8C_playerFv |
| 0x001b4a40 | 1484 | GLOBAL | C_player | Do_AB_OwnerCar | Do_AB_OwnerCar__8C_playerFv |
| 0x001b5010 | 140 | GLOBAL | C_player | NastavTypKrizkuPodleZbraneAtToTiSuliniOdSebePoznaji | NastavTypKrizkuPodleZbraneAtToTiSuliniOdSebePoznaji__8C_playerFv |
| 0x001b50a0 | 304 | GLOBAL | C_player | IndicatorSetAimed | IndicatorSetAimed__8C_playerFv |
| 0x001b51d0 | 220 | GLOBAL | C_player | ShootDeAimed | ShootDeAimed__8C_playerFP15S_GameItemTable |
| 0x001b52b0 | 356 | GLOBAL | C_player | SetSniperMode | SetSniperMode__8C_playerFb |
| 0x001b5420 | 180 | GLOBAL | C_player | _intern_ChangeModel | _intern_ChangeModel__8C_playerFv |
| 0x001b54e0 | 2000 | GLOBAL | C_player | Can_UseCar | Can_UseCar__8C_playerFP5C_cari |
| 0x001b5cb0 | 72 | GLOBAL | C_player | SetCarDamage | SetCarDamage__8C_playerFP5C_car |
| 0x001b5d00 | 92 | GLOBAL | C_player | InvalidateCarDamage | InvalidateCarDamage__8C_playerFP5C_car |
| 0x001b5d60 | 336 | GLOBAL | C_player | AddAlphaObject | AddAlphaObject__8C_playerFP9I3D_frame |
| 0x001b5eb0 | 572 | 13 | C_player::S_hlp$8169C_player_cpp | cbAlpha | cbAlpha__Q28C_player22S_hlp$8169C_player_cppFP9I3D_frameRQ23std54vector<P10I3D_object,Q23std24allocator<P10I3D_object>> |
| 0x001b60f0 | 8 | GLOBAL | C_player | DelAlphaObject | DelAlphaObject__8C_playerFP9I3D_frame |
| 0x001b6100 | 136 | GLOBAL | C_player | SetAlpha | SetAlpha__8C_playerFf |
| 0x001b6190 | 8 | GLOBAL | C_player | ShowWeaponName | ShowWeaponName__8C_playerFv |
| 0x001b61a0 | 656 | GLOBAL | C_player | LockControls | LockControls__8C_playerFb |
| 0x001b6430 | 20 | GLOBAL | C_player | FadeReset | FadeReset__8C_playerFv |
| 0x001b6450 | 160 | GLOBAL | C_program | {ctor} | __ct__9C_programFv |
| 0x001b64f0 | 16 | 13 | Metrowerks::details::compressed_pair_imp<Q23std38allocator<Q29C_program13S_instruction>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details72compressed_pair_imp<Q23std38allocator<Q29C_program13S_instruction>,Ui,1>FUi |
| 0x001b6500 | 204 | GLOBAL | C_program | {dtor} | __dt__9C_programFv |
| 0x001b65d0 | 364 | GLOBAL | C_program | SaveSourceCode | SaveSourceCode__9C_programFP7C_chunk |
| 0x001b6740 | 896 | GLOBAL | C_program | LoadSourceCode | LoadSourceCode__9C_programFP7C_chunk |
| 0x001b6ac0 | 160 | GLOBAL |  | EqualFinalPath__FPCcPCc | EqualFinalPath__FPCcPCc |
| 0x001b6b60 | 524 | GLOBAL | C_program | TryChangeProgramData | TryChangeProgramData__9C_programFv |
| 0x001b6d70 | 124 | GLOBAL | C_program | SetSourceCode | SetSourceCode__9C_programFPc |
| 0x001b6df0 | 308 | GLOBAL | C_program | Init | Init__9C_programFv |
| 0x001b6f30 | 428 | GLOBAL | C_program | Done | Done__9C_programFv |
| 0x001b70e0 | 132 | GLOBAL | C_program | DuplicateProgramFrom | DuplicateProgramFrom__9C_programFP9C_program |
| 0x001b7170 | 424 | GLOBAL | C_program | ExtrudeParams | ExtrudeParams__9C_programFPcRiPPc |
| 0x001b7320 | 316 | GLOBAL | C_program | DecodeOperand | DecodeOperand__9C_programFPcRiRf |
| 0x001b7460 | 304 | GLOBAL | C_program | DecodeOperandInt | DecodeOperandInt__9C_programFPcRiRi |
| 0x001b7590 | 156 | GLOBAL | C_program | ExtrudeString | ExtrudeString__9C_programFPc |
| 0x001b7630 | 520 | GLOBAL | C_program | DecodeLet | DecodeLet__9C_programFRQ29C_program13S_instructionPc |
| 0x001b7840 | 292 | GLOBAL | C_program | DecodeIf | DecodeIf__9C_programFRQ29C_program13S_instructionPc |
| 0x001b7970 | 9352 | GLOBAL | C_program | DecodeParams | DecodeParams__9C_programFPcRQ29C_program13S_instruction |
| 0x001b9e00 | 380 | GLOBAL | C_program | DecodeInstruction | DecodeInstruction__9C_programFPcRi |
| 0x001b9f80 | 696 | GLOBAL | C_program | Compile | Compile__9C_programFPcbPiPi |
| 0x001ba240 | 8 | GLOBAL | C_program | DecompileCommand | DecompileCommand__9C_programFiPc |
| 0x001ba250 | 60 | GLOBAL | C_program | GetFloatReg | GetFloatReg__9C_programFi |
| 0x001ba290 | 60 | GLOBAL | C_program | GetFrameReg | GetFrameReg__9C_programFi |
| 0x001ba2d0 | 60 | GLOBAL | C_program | GetActorReg | GetActorReg__9C_programFi |
| 0x001ba310 | 48 | GLOBAL | C_program | SetFloatReg | SetFloatReg__9C_programFif |
| 0x001ba340 | 48 | GLOBAL | C_program | SetFrameReg | SetFrameReg__9C_programFiP9I3D_frame |
| 0x001ba370 | 48 | GLOBAL | C_program | SetActorReg | SetActorReg__9C_programFiP7C_actor |
| 0x001ba3a0 | 236 | GLOBAL | C_program | NextCommand | NextCommand__9C_programFPc |
| 0x001ba490 | 52 | GLOBAL | C_program | CallSubroutine | CallSubroutine__9C_programFv |
| 0x001ba4d0 | 236 | GLOBAL | C_program | LeaveCompactBlock | LeaveCompactBlock__9C_programFb |
| 0x001ba5c0 | 144 | GLOBAL | C_program | CallProcess | CallProcess__9C_programFUi |
| 0x001ba650 | 252 | GLOBAL | C_program | ClearAfterEnd | ClearAfterEnd__9C_programFv |
| 0x001ba750 | 384 | GLOBAL | C_program | SetEvent | SetEvent__9C_programFi |
| 0x001ba8d0 | 116 | GLOBAL | C_program | IsEventID | IsEventID__9C_programFiPi |
| 0x001ba950 | 64 | GLOBAL | C_program | InvalidateActorReg | InvalidateActorReg__9C_programFP7C_actor |
| 0x001ba990 | 320 | GLOBAL | C_program | SaveGameGetSize | SaveGameGetSize__9C_programFv |
| 0x001baad0 | 624 | GLOBAL | C_program | SaveGameSave | SaveGameSave__9C_programFPPv |
| 0x001bad40 | 788 | GLOBAL | C_program | SaveGameLoad | SaveGameLoad__9C_programFPPv |
| 0x001bb060 | 108 | GLOBAL | C_program | FindActorLater | FindActorLater__9C_programFiUi |
| 0x001bb0d0 | 95704 | GLOBAL | C_program | Process | Process__9C_programFUi |
| 0x001d26b0 | 780 | 13 | gif::vector<P7C_actor,2> | push_back | push_back__Q23gif19vector<P7C_actor,2>FRCP7C_actor |
| 0x001d29c0 | 364 | 13 | C_program::S_help$4741C_program_cpp | SwitchLightMap | SwitchLightMap__Q29C_program24S_help$4741C_program_cppFP9I3D_framei |
| 0x001d2b30 | 24 | 13 | std::__vector_deleter<Q29C_program13S_instruction,Q23std38allocator<Q29C_program13S_instruction>> | {ctor} | __ct__Q23std92__vector_deleter<Q29C_program13S_instruction,Q23std38allocator<Q29C_program13S_instruction>>Fv |
| 0x001d2b50 | 116 | GLOBAL | C_program | GetInterCityName | GetInterCityName__9C_programFPc |
| 0x001d2bd0 | 4304 | GLOBAL | C_program | PreloadAnims | PreloadAnims__9C_programFv |
| 0x001d3ca0 | 376 | GLOBAL | C_program | DebilitizeScripts | DebilitizeScripts__9C_programFv |
| 0x001d3e20 | 232 | 13 | C_program::S_hlp$18127C_program_cpp | cbIsFrameInHierarchy | cbIsFrameInHierarchy__Q29C_program24S_hlp$18127C_program_cppFP9I3D_frameP9I3D_frame |
| 0x001d3f10 | 784 | WEAK | std::__vector_pod<Us,Q23std13allocator<Us>> | insert | insert__Q23std38__vector_pod<Us,Q23std13allocator<Us>>FPUsUiRCUs |
| 0x001d4220 | 120 | WEAK | std::__vector_pod<Us,Q23std13allocator<Us>> | resize | resize__Q23std38__vector_pod<Us,Q23std13allocator<Us>>FUiRCUs |
| 0x001d42a0 | 1124 | WEAK | std::__vector_imp<Q29C_program13S_instruction,Q23std38allocator<Q29C_program13S_instruction>,0> | insert | insert__Q23std90__vector_imp<Q29C_program13S_instruction,Q23std38allocator<Q29C_program13S_instruction>,0>FPQ29C_program13S_instructionUiRCQ29C_program13S_instruction |
| 0x001d4710 | 24 | 13 | Metrowerks::details::compressed_pair_imp<Q23std38allocator<Q29C_program13S_instruction>,Ui,1> | swap | swap__Q310Metrowerks7details72compressed_pair_imp<Q23std38allocator<Q29C_program13S_instruction>,Ui,1>FRQ310Metrowerks7details72compressed_pair_imp<Q23std38allocator<Q29C_program13S_instruction>,Ui,1> |
| 0x001d4730 | 68 | WEAK | std::__vector_deleter<Q29C_program13S_instruction,Q23std38allocator<Q29C_program13S_instruction>> | clear | clear__Q23std92__vector_deleter<Q29C_program13S_instruction,Q23std38allocator<Q29C_program13S_instruction>>Fv |
| 0x001d4780 | 116 | WEAK | std::__vector_deleter<Q29C_program13S_instruction,Q23std38allocator<Q29C_program13S_instruction>> | {dtor} | __dt__Q23std92__vector_deleter<Q29C_program13S_instruction,Q23std38allocator<Q29C_program13S_instruction>>Fv |
| 0x001d4800 | 84 | WEAK | std::__vector_deleter<16ai_character_log,Q23std29allocator<16ai_character_log>> | clear | clear__Q23std74__vector_deleter<16ai_character_log,Q23std29allocator<16ai_character_log>>Fv |
| 0x001d4860 | 68 | WEAK | std::__vector_deleter<Q29C_program13S_prog_change,Q23std38allocator<Q29C_program13S_prog_change>> | clear | clear__Q23std92__vector_deleter<Q29C_program13S_prog_change,Q23std38allocator<Q29C_program13S_prog_change>>Fv |
| 0x001d48b0 | 188 | WEAK | std::__vector_pod<Us,Q23std13allocator<Us>> | erase | erase__Q23std38__vector_pod<Us,Q23std13allocator<Us>>FPUsPUs |
| 0x001d4970 | 116 | WEAK | std::__vector_deleter<Q29C_program13S_prog_change,Q23std38allocator<Q29C_program13S_prog_change>> | {dtor} | __dt__Q23std92__vector_deleter<Q29C_program13S_prog_change,Q23std38allocator<Q29C_program13S_prog_change>>Fv |
| 0x001d49f0 | 108 | 13 | std::vector<Q29C_program13S_prog_change,Q23std38allocator<Q29C_program13S_prog_change>> | {dtor} | __dt__Q23std82vector<Q29C_program13S_prog_change,Q23std38allocator<Q29C_program13S_prog_change>>Fv |
| 0x001d4a60 | 16 | 13 | Metrowerks::details::compressed_pair_imp<Q23std38allocator<Q29C_program13S_prog_change>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details72compressed_pair_imp<Q23std38allocator<Q29C_program13S_prog_change>,Ui,1>FUi |
| 0x001d4a70 | 60 | GLOBAL | C_public_physics | {ctor} | __ct__16C_public_physicsFv |
| 0x001d4ab0 | 96 | GLOBAL | C_public_physics | {dtor} | __dt__16C_public_physicsFv |
| 0x001d4b10 | 8 | GLOBAL | C_public_physics | AI | AI__16C_public_physicsFUi |
| 0x001d4b20 | 8 | GLOBAL | C_public_physics | Update | Update__16C_public_physicsFUi |
| 0x001d4b30 | 8 | GLOBAL | C_public_physics | NetDirect | NetDirect__16C_public_physicsFUi |
| 0x001d4b40 | 8 | GLOBAL | C_public_physics | Init | Init__16C_public_physicsFP9I3D_frame |
| 0x001d4b50 | 8 | GLOBAL | C_public_physics | DuplicateFrom | DuplicateFrom__16C_public_physicsFP7C_actor |
| 0x001d4b60 | 128 | GLOBAL | C_public_physics | ChangeState | ChangeState__16C_public_physicsFv |
| 0x001d4be0 | 60 | GLOBAL | C_public_physics | GameInit | GameInit__16C_public_physicsFv |
| 0x001d4c20 | 56 | GLOBAL | C_public_physics | GameDone | GameDone__16C_public_physicsFv |
| 0x001d4c60 | 8 | GLOBAL | C_public_physics | Collision | Collision__16C_public_physicsFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001d4c70 | 8 | GLOBAL | C_public_physics | Hit | Hit__16C_public_physicsF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x001d4c80 | 8 | 13 | C_public_physics | SaveData | SaveData__16C_public_physicsFP7C_chunk |
| 0x001d4c90 | 8 | 13 | C_public_physics | LoadData | LoadData__16C_public_physicsFP7C_chunk |
| 0x001d4ca0 | 588 | GLOBAL | C_pumper | {ctor} | __ct__8C_pumperFv |
| 0x001d4ef0 | 112 | GLOBAL | C_pumper | {dtor} | __dt__8C_pumperFv |
| 0x001d4f60 | 44 | GLOBAL | C_pumper | Init | Init__8C_pumperFP9I3D_frame |
| 0x001d4f90 | 288 | GLOBAL | C_pumper | SaveData | SaveData__8C_pumperFP7C_chunk |
| 0x001d50b0 | 744 | GLOBAL | C_pumper | LoadData | LoadData__8C_pumperFP7C_chunk |
| 0x001d53a0 | 40 | GLOBAL | C_pumper | DuplicateFrom | DuplicateFrom__8C_pumperFP7C_actor |
| 0x001d53d0 | 10708 | GLOBAL | C_pumper | GameInit | GameInit__8C_pumperFv |
| 0x001d7db0 | 676 | GLOBAL | C_pumper | GameDone | GameDone__8C_pumperFv |
| 0x001d8060 | 212 | GLOBAL | C_pumper | Collision | Collision__8C_pumperFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001d8140 | 232 | GLOBAL | C_pumper | Hit | Hit__8C_pumperF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x001d8230 | 8 | GLOBAL | C_pumper | Death | Death__8C_pumperFv |
| 0x001d8240 | 2656 | GLOBAL | C_pumper | AI | AI__8C_pumperFUi |
| 0x001d8ca0 | 780 | GLOBAL | C_pumper | AI_walk | AI_walk__8C_pumperFUi |
| 0x001d8fb0 | 512 | GLOBAL | C_pumper | AI_turn | AI_turn__8C_pumperFUiRC8S_vector |
| 0x001d91b0 | 836 | GLOBAL | C_pumper | Update | Update__8C_pumperFUi |
| 0x001d9500 | 208 | GLOBAL | C_pumper | ground | ground__8C_pumperFv |
| 0x001d95d0 | 176 | GLOBAL | C_pumper | SetAnim | SetAnim__8C_pumperFf |
| 0x001d9680 | 1968 | GLOBAL | C_pumper | ChooseNextAction | ChooseNextAction__8C_pumperFv |
| 0x001d9e30 | 140 | GLOBAL | C_pumper | NextNode | NextNode__8C_pumperFv |
| 0x001d9ec0 | 800 | GLOBAL | C_pumper | Start | Start__8C_pumperFP5C_car |
| 0x001da1e0 | 972 | GLOBAL | C_pumper | CanPump | CanPump__8C_pumperFP5C_car |
| 0x001da5b0 | 864 | GLOBAL |  | pumper_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi | pumper_notify_callback__FP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi |
| 0x001da910 | 88 | GLOBAL | C_pumper | StartPumpFuel | StartPumpFuel__8C_pumperFv |
| 0x001da970 | 92 | GLOBAL | C_pumper | EndPumpFuel | EndPumpFuel__8C_pumperFv |
| 0x001da9d0 | 76 | GLOBAL | C_pumper | InitSounds | InitSounds__8C_pumperFv |
| 0x001daa20 | 276 | GLOBAL | C_pumper | InitSound | InitSound__8C_pumperFUi |
| 0x001dab40 | 116 | GLOBAL | C_pumper | ReleaseSounds | ReleaseSounds__8C_pumperFv |
| 0x001dabc0 | 164 | GLOBAL | C_pumper | GetSpecAnimName | GetSpecAnimName__8C_pumperF18E_anim_pumper_spec |
| 0x001dac70 | 484 | GLOBAL | C_pumper | PumperPanic | PumperPanic__8C_pumperFv |
| 0x001dae60 | 32 | GLOBAL | C_pumper | SaveGameGetSize | SaveGameGetSize__8C_pumperFv |
| 0x001dae80 | 716 | GLOBAL | C_pumper | SaveGameSave | SaveGameSave__8C_pumperFPPv |
| 0x001db150 | 948 | GLOBAL | C_pumper | SaveGameLoad | SaveGameLoad__8C_pumperFPPv |
| 0x001db510 | 68 | GLOBAL | C_pumper | PlaySound | PlaySound__8C_pumperFUib |
| 0x001db560 | 96 | GLOBAL | C_pumper | ReleaseColls | ReleaseColls__8C_pumperFv |
| 0x001db5c0 | 60 | GLOBAL | C_pumper | IsPumping | IsPumping__8C_pumperFv |
| 0x001db600 | 16 | GLOBAL | C_pumper | IsInPanic | IsInPanic__8C_pumperFv |
| 0x001db610 | 120 | GLOBAL | C_pumper | ScriptCanPump | ScriptCanPump__8C_pumperFUi |
| 0x001db690 | 60 | GLOBAL | C_racecam | {ctor} | __ct__9C_racecamFv |
| 0x001db6d0 | 96 | GLOBAL | C_racecam | {dtor} | __dt__9C_racecamFv |
| 0x001db730 | 112 | GLOBAL | C_racecam | Init | Init__9C_racecamFP9I3D_frame |
| 0x001db7a0 | 64 | GLOBAL | C_racecam | SaveData | SaveData__9C_racecamFP7C_chunk |
| 0x001db7e0 | 176 | GLOBAL | C_racecam | LoadData | LoadData__9C_racecamFP7C_chunk |
| 0x001db890 | 96 | GLOBAL | C_racecam | DuplicateFrom | DuplicateFrom__9C_racecamFP7C_actor |
| 0x001db8f0 | 104 | GLOBAL | C_racecam | GameInit | GameInit__9C_racecamFv |
| 0x001db960 | 56 | GLOBAL | C_racecam | GameDone | GameDone__9C_racecamFv |
| 0x001db9a0 | 8 | 13 | C_racecam | SaveGameGetSize | SaveGameGetSize__9C_racecamFv |
| 0x001db9b0 | 8 | 13 | C_racecam | SaveGameSave | SaveGameSave__9C_racecamFPPv |
| 0x001db9c0 | 8 | 13 | C_racecam | SaveGameLoad | SaveGameLoad__9C_racecamFPPv |
| 0x001db9d0 | 60 | GLOBAL | C_racing_osegment | {ctor} | __ct__17C_racing_osegmentFv |
| 0x001dba10 | 1068 | GLOBAL | C_racing | CreateSegments | CreateSegments__8C_racingFv |
| 0x001dbe40 | 8 | 13 | C_circuit | GetRightPath | GetRightPath__9C_circuitFv |
| 0x001dbe50 | 16 | 13 | std::vector<P10C_web_node,Q23std24allocator<P10C_web_node>> | operator[] | __vc__Q23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>>FUi |
| 0x001dbe60 | 8 | 13 | C_circuit | GetLeftPath | GetLeftPath__9C_circuitFv |
| 0x001dbe70 | 1332 | GLOBAL | C_racing | LoadSegments | LoadSegments__8C_racingFbPcUi |
| 0x001dc3b0 | 244 | GLOBAL | C_racing | ClearSegments | ClearSegments__8C_racingFUi |
| 0x001dc4b0 | 72 | GLOBAL | C_racing | GetSegments | GetSegments__8C_racingFUiUi |
| 0x001dc500 | 256 | GLOBAL | C_racing | GetSegmentTotalTime | GetSegmentTotalTime__8C_racingFUiUi |
| 0x001dc600 | 68 | GLOBAL | C_rail_generator | GetCheckpoints | GetCheckpoints__16C_rail_generatorFb |
| 0x001dc650 | 96 | GLOBAL | C_rail_generator | {ctor} | __ct__16C_rail_generatorFv |
| 0x001dc6b0 | 96 | GLOBAL | C_rail_generator | {dtor} | __dt__16C_rail_generatorFv |
| 0x001dc710 | 56 | GLOBAL | C_rail_generator | Init | Init__16C_rail_generatorFP9I3D_frame |
| 0x001dc750 | 208 | GLOBAL | C_rail_generator | SaveData | SaveData__16C_rail_generatorFP7C_chunk |
| 0x001dc820 | 512 | GLOBAL | C_rail_generator | LoadData | LoadData__16C_rail_generatorFP7C_chunk |
| 0x001dca20 | 8 | GLOBAL | C_rail_generator | DuplicateFrom | DuplicateFrom__16C_rail_generatorFP7C_actor |
| 0x001dca30 | 5396 | GLOBAL | C_rail_generator | GameInit | GameInit__16C_rail_generatorFv |
| 0x001ddf50 | 588 | GLOBAL | C_rail_generator | GameDone | GameDone__16C_rail_generatorFv |
| 0x001de1a0 | 688 | GLOBAL | C_rail_generator | AI | AI__16C_rail_generatorFUi |
| 0x001de450 | 8 | GLOBAL | C_rail_generator | Update | Update__16C_rail_generatorFUi |
| 0x001de460 | 480 | GLOBAL | C_rail_generator | GetNearFreeTrackIDandNodeInRange | GetNearFreeTrackIDandNodeInRange__16C_rail_generatorFPP10C_web_node |
| 0x001de640 | 16 | 13 | std::__vector_imp<P10C_web_node,Q23std24allocator<P10C_web_node>,1> | operator[] | __vc__Q23std62__vector_imp<P10C_web_node,Q23std24allocator<P10C_web_node>,1>FUi |
| 0x001de650 | 200 | GLOBAL | C_rail_generator | GetNextTrackNode | GetNextTrackNode__16C_rail_generatorFP10C_web_nodeb |
| 0x001de720 | 200 | GLOBAL | C_rail_generator | GetPrevTrackNode | GetPrevTrackNode__16C_rail_generatorFP10C_web_nodeb |
| 0x001de7f0 | 420 | GLOBAL | C_rail_generator | GetNodeOutOfRange | GetNodeOutOfRange__16C_rail_generatorFbP10C_web_nodeb |
| 0x001de9a0 | 88 | GLOBAL | C_rail_generator | GetFreeTram | GetFreeTram__16C_rail_generatorFv |
| 0x001dea00 | 88 | GLOBAL | C_rail_generator | GetFreeMetro | GetFreeMetro__16C_rail_generatorFv |
| 0x001dea60 | 592 | GLOBAL | C_rail_generator | GetFreeRailHuman | GetFreeRailHuman__16C_rail_generatorFv |
| 0x001decb0 | 80 | GLOBAL | C_rail_generator | ISFree | ISFree__16C_rail_generatorFP9C_railway |
| 0x001ded00 | 400 | GLOBAL | C_rail_generator | ChangeSceneInit | ChangeSceneInit__16C_rail_generatorFv |
| 0x001dee90 | 116 | GLOBAL | C_rail_generator | ChangeSceneDone | ChangeSceneDone__16C_rail_generatorFv |
| 0x001def10 | 84 | GLOBAL | C_rail_generator | GetMetroTracksCount | GetMetroTracksCount__16C_rail_generatorFv |
| 0x001def70 | 84 | GLOBAL | C_rail_generator | GetTramTracksCount | GetTramTracksCount__16C_rail_generatorFv |
| 0x001defd0 | 816 | GLOBAL | C_rail_generator | CreateTracks | CreateTracks__16C_rail_generatorFPQ23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>>b |
| 0x001df300 | 1388 | GLOBAL | C_rail_generator | InitTracks | InitTracks__16C_rail_generatorFb |
| 0x001df870 | 304 | GLOBAL | C_rail_generator | DoneTracks | DoneTracks__16C_rail_generatorFb |
| 0x001df9a0 | 140 | 13 | std::vector<P9C_railway,Q23std22allocator<P9C_railway>> | {dtor} | __dt__Q23std50vector<P9C_railway,Q23std22allocator<P9C_railway>>Fv |
| 0x001dfa30 | 140 | 13 | std::vector<P12S_rail_track,Q23std26allocator<P12S_rail_track>> | {dtor} | __dt__Q23std58vector<P12S_rail_track,Q23std26allocator<P12S_rail_track>>Fv |
| 0x001dfac0 | 140 | 13 | std::vector<P12S_rail_human,Q23std26allocator<P12S_rail_human>> | {dtor} | __dt__Q23std58vector<P12S_rail_human,Q23std26allocator<P12S_rail_human>>Fv |
| 0x001dfb50 | 488 | GLOBAL | S_RailSeats | FindFreeSeat | FindFreeSeat__11S_RailSeatsFUiP8S_vectorf |
| 0x001dfd40 | 148 | GLOBAL | S_RailSeats | NumFreeSeats | NumFreeSeats__11S_RailSeatsFi |
| 0x001dfde0 | 580 | GLOBAL | S_RailSeats | GetSeatPos | GetSeatPos__11S_RailSeatsFUi |
| 0x001e0030 | 124 | GLOBAL | S_RailSeats | GetSeatDir | GetSeatDir__11S_RailSeatsFUi |
| 0x001e00b0 | 172 | GLOBAL | S_RailSeats | EmptySeat | EmptySeat__11S_RailSeatsFUiPv |
| 0x001e0160 | 180 | GLOBAL | S_RailSeats | ReserveSeat | ReserveSeat__11S_RailSeatsFUiPv |
| 0x001e0220 | 344 | GLOBAL | S_RailSeats | FullSeat | FullSeat__11S_RailSeatsFUiPv |
| 0x001e0380 | 208 | GLOBAL | S_RailSeats | AddRow | AddRow__11S_RailSeatsFP13rail_seat_row |
| 0x001e0450 | 8 | 13 | rail_seat_row | {ctor} | __ct__13rail_seat_rowFv |
| 0x001e0460 | 296 | GLOBAL | S_RailSeats | Duplicate | Duplicate__11S_RailSeatsFUi |
| 0x001e0590 | 1548 | GLOBAL | S_RailSeats | Init | Init__11S_RailSeatsFPv |
| 0x001e0ba0 | 140 | GLOBAL | S_RailSeats | Destroy | Destroy__11S_RailSeatsFv |
| 0x001e0c30 | 52 | GLOBAL | S_rail_human | {ctor} | __ct__12S_rail_humanFv |
| 0x001e0c70 | 440 | GLOBAL | S_rail_human | RotateTo | RotateTo__12S_rail_humanFR8S_vectorUi |
| 0x001e0e30 | 3980 | GLOBAL | S_rail_human | MoveTo | MoveTo__12S_rail_humanFR8S_vectorUi |
| 0x001e1dc0 | 284 | GLOBAL | S_rail_human | SetAnim | SetAnim__12S_rail_humanFUi |
| 0x001e1ee0 | 204 | GLOBAL | S_rail_human | Kill | Kill__12S_rail_humanFP9C_railway |
| 0x001e1fb0 | 740 | GLOBAL | S_rail_human | CreateColl | CreateColl__12S_rail_humanFP9C_railway |
| 0x001e22a0 | 228 | GLOBAL | S_rail_human | UpdateColl | UpdateColl__12S_rail_humanFv |
| 0x001e2390 | 80 | GLOBAL | S_rail_human | Done | Done__12S_rail_humanFv |
| 0x001e23e0 | 196 | GLOBAL | S_rail_human | DoPanic | DoPanic__12S_rail_humanFUi |
| 0x001e24b0 | 5584 | GLOBAL | C_railway | trafficAI | trafficAI__9C_railwayFUi |
| 0x001e3a80 | 164 | GLOBAL | S_rail_human | IsWoman | IsWoman__12S_rail_humanFv |
| 0x001e3b30 | 148 | GLOBAL | C_railway | WantLeave | WantLeave__9C_railwayFv |
| 0x001e3bd0 | 124 | GLOBAL | C_railway | WantEnter | WantEnter__9C_railwayFv |
| 0x001e3c50 | 532 | GLOBAL | C_railway | {ctor} | __ct__9C_railwayFv |
| 0x001e3e70 | 104 | 13 | std::__vector_imp<Ui,Q23std13allocator<Ui>,1> | {dtor} | __dt__Q23std40__vector_imp<Ui,Q23std13allocator<Ui>,1>Fv |
| 0x001e3ee0 | 360 | GLOBAL | C_railway | {dtor} | __dt__9C_railwayFv |
| 0x001e4050 | 76 | GLOBAL | C_railway | Init | Init__9C_railwayFP9I3D_frame |
| 0x001e40a0 | 64 | GLOBAL | C_railway | DuplicateFrom | DuplicateFrom__9C_railwayFP7C_actor |
| 0x001e40e0 | 328 | GLOBAL | C_railway | SaveData | SaveData__9C_railwayFP7C_chunk |
| 0x001e4230 | 860 | GLOBAL | C_railway | LoadData | LoadData__9C_railwayFP7C_chunk |
| 0x001e4590 | 620 | GLOBAL | C_railway | CreateBackVector | CreateBackVector__9C_railwayFR8S_vector |
| 0x001e4800 | 24 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | back | back__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>Fv |
| 0x001e4820 | 1224 | GLOBAL | C_railway | CreateForwardVector | CreateForwardVector__9C_railwayFUif |
| 0x001e4cf0 | 64 | GLOBAL | C_railway | IsStopPoint | IsStopPoint__9C_railwayFUi |
| 0x001e4d30 | 524 | GLOBAL | C_railway | IsSemaPoint | IsSemaPoint__9C_railwayFUi |
| 0x001e4f40 | 864 | GLOBAL | C_railway | MustStop | MustStop__9C_railwayFPUi |
| 0x001e52a0 | 1908 | GLOBAL | C_railway | FindBackPos | FindBackPos__9C_railwayFR8S_vectorfPi |
| 0x001e5a20 | 16 | 13 | std::__vector_imp<Ui,Q23std13allocator<Ui>,1> | operator[] | __vc__Q23std40__vector_imp<Ui,Q23std13allocator<Ui>,1>FUi |
| 0x001e5a30 | 1040 | GLOBAL | C_railway | SpringProcess | SpringProcess__9C_railwayFUiiR8S_vectorR8S_vectorR8S_vectorR8S_vector |
| 0x001e5e40 | 6212 | GLOBAL | C_railway | AI | AI__9C_railwayFUi |
| 0x001e7690 | 1040 | GLOBAL | C_railway | OutOfRangeDeactivate | OutOfRangeDeactivate__9C_railwayFv |
| 0x001e7aa0 | 420 | GLOBAL | C_railway | InRangeActivate | InRangeActivate__9C_railwayFv |
| 0x001e7c50 | 2996 | GLOBAL | C_railway | Update | Update__9C_railwayFUi |
| 0x001e8810 | 8 | GLOBAL | C_railway | NetDirect | NetDirect__9C_railwayFUi |
| 0x001e8820 | 10076 | GLOBAL | C_railway | GameInit | GameInit__9C_railwayFv |
| 0x001eaf80 | 52 | 13 | std::vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>> | {ctor} | __ct__Q23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>Fv |
| 0x001eafc0 | 8 | 13 | std::allocator<Ui> | {ctor} | __ct__Q23std13allocator<Ui>Fv |
| 0x001eafd0 | 48 | 13 | tDynamicCollObject | {ctor} | __ct__18tDynamicCollObjectFv |
| 0x001eb000 | 1504 | GLOBAL | C_railway | GameDone | GameDone__9C_railwayFv |
| 0x001eb5e0 | 900 | GLOBAL | C_railway | Collision | Collision__9C_railwayFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x001eb970 | 3068 | GLOBAL | C_railway | IsPossibleColl | IsPossibleColl__9C_railwayFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor |
| 0x001ec570 | 3640 | GLOBAL | C_railway | FillCurve | FillCurve__9C_railwayFPQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8> |
| 0x001ed3b0 | 1692 | GLOBAL | C_railway | PreduceMovementCar | PreduceMovementCar__9C_railwayFP5C_carbRQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8> |
| 0x001eda50 | 2660 | GLOBAL | C_railway | PreduceMovement | PreduceMovement__9C_railwayFv |
| 0x001ee4c0 | 696 | GLOBAL | C_railway | Place | Place__9C_railwayFiP10C_web_node |
| 0x001ee780 | 2532 | GLOBAL | C_railway | Place | Place__9C_railwayFi |
| 0x001ef170 | 24 | 13 | std::__vector_imp<Ui,Q23std13allocator<Ui>,1> | back | back__Q23std40__vector_imp<Ui,Q23std13allocator<Ui>,1>Fv |
| 0x001ef190 | 1484 | GLOBAL | C_railway | Collide | Collide__9C_railwayFv |
| 0x001ef760 | 2640 | GLOBAL | C_railway | Hit | Hit__9C_railwayF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x001f01b0 | 5588 | GLOBAL | C_railway | GetSafeSpeed | GetSafeSpeed__9C_railwayFRf |
| 0x001f1790 | 796 | GLOBAL | C_railway | AddPerson | AddPerson__9C_railwayFP9I3D_modelii |
| 0x001f1ab0 | 656 | GLOBAL | C_railway | RemovePerson | RemovePerson__9C_railwayFP12S_rail_humanUi |
| 0x001f1d40 | 248 | GLOBAL | C_railway | GetNearestVagon | GetNearestVagon__9C_railwayFP8S_vector |
| 0x001f1e40 | 268 | 13 | C_railway | CheckDistance | CheckDistance__9C_railwayFR8S_vector |
| 0x001f1f50 | 2416 | GLOBAL | C_railway | Enumerate | Enumerate__9C_railwayFv |
| 0x001f28c0 | 8 | 13 | std::__vector_deleter<9raildummy,Q23std21allocator<9raildummy>> | data | data__Q23std58__vector_deleter<9raildummy,Q23std21allocator<9raildummy>>Fv |
| 0x001f28d0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std21allocator<9raildummy>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details55compressed_pair_imp<Q23std21allocator<9raildummy>,Ui,1>FUi |
| 0x001f28e0 | 1508 | GLOBAL |  | cbEnumDummies__FP9I3D_frameUi | cbEnumDummies__FP9I3D_frameUi |
| 0x001f2ed0 | 152 | GLOBAL |  | cbEnumDummiesVagons__FP9I3D_frameUi | cbEnumDummiesVagons__FP9I3D_frameUi |
| 0x001f2f70 | 248 | GLOBAL |  | cbEnumLights__FP9I3D_frameUi | cbEnumLights__FP9I3D_frameUi |
| 0x001f3070 | 40 | GLOBAL |  | cbEnumDestroy__FP9I3D_frameUi | cbEnumDestroy__FP9I3D_frameUi |
| 0x001f30a0 | 192 | GLOBAL | rail_entries | AddEntry | AddEntry__12rail_entriesFP10rail_entry |
| 0x001f3160 | 8 | 13 | rail_entry | {ctor} | __ct__10rail_entryFv |
| 0x001f3170 | 48 | GLOBAL | rail_entries | Destroy | Destroy__12rail_entriesFv |
| 0x001f31a0 | 100 | GLOBAL | C_railway | DestroyEntries | DestroyEntries__9C_railwayFv |
| 0x001f3210 | 896 | GLOBAL | C_railway | GenEntries | GenEntries__9C_railwayFv |
| 0x001f3590 | 80 | 13 | C_using_object::S_using_object | {ctor} | __ct__Q214C_using_object14S_using_objectFv |
| 0x001f35e0 | 124 | GLOBAL | C_railway | AddOuterEntries | AddOuterEntries__9C_railwayFv |
| 0x001f3660 | 124 | GLOBAL | C_railway | AddInnerEntries | AddInnerEntries__9C_railwayFv |
| 0x001f36e0 | 124 | GLOBAL | C_railway | RemoveOuterEntries | RemoveOuterEntries__9C_railwayFv |
| 0x001f3760 | 124 | GLOBAL | C_railway | RemoveInnerEntries | RemoveInnerEntries__9C_railwayFv |
| 0x001f37e0 | 1444 | GLOBAL | C_railway | GetOuterEntry | GetOuterEntry__9C_railwayFP7C_actorP8S_vectorP8S_vectorPUi |
| 0x001f3d90 | 1040 | GLOBAL | C_railway | GetOuterEntry | GetOuterEntry__9C_railwayFUiUiP8S_vectorP8S_vector |
| 0x001f41a0 | 716 | GLOBAL | C_railway | GetInnerEntry | GetInnerEntry__9C_railwayFP7C_actorP8S_vectorP8S_vectorPUi |
| 0x001f4470 | 104 | GLOBAL | C_railway | GetEntry | GetEntry__9C_railwayFP7C_actorP8S_vectorP8S_vectorPUi |
| 0x001f44e0 | 1480 | GLOBAL | C_railway | GetOuterEntry | GetOuterEntry__9C_railwayFRC8S_vectorP8S_vectorP8S_vectorPUi |
| 0x001f4ab0 | 584 | GLOBAL | C_railway | GetInnerEntry | GetInnerEntry__9C_railwayFRC8S_vectorP8S_vectorP8S_vectorPUi |
| 0x001f4d00 | 1452 | GLOBAL | C_railway | ExportEntries | ExportEntries__9C_railwayFv |
| 0x001f52b0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6rentry>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6rentry>,Ui,1>FUi |
| 0x001f52c0 | 28 | GLOBAL | rail_doors | Open | Open__10rail_doorsFv |
| 0x001f52e0 | 24 | GLOBAL | rail_doors | Close | Close__10rail_doorsFv |
| 0x001f5300 | 332 | GLOBAL | rail_doors | Place | Place__10rail_doorsFv |
| 0x001f5450 | 192 | GLOBAL | rail_doors | Tick | Tick__10rail_doorsFi |
| 0x001f5510 | 244 | GLOBAL | rail_doors | AddFrame | AddFrame__10rail_doorsFP9I3D_frameb |
| 0x001f5610 | 68 | GLOBAL | rail_doors | Init | Init__10rail_doorsFv |
| 0x001f5660 | 72 | GLOBAL | rail_doors | Destroy | Destroy__10rail_doorsFv |
| 0x001f56b0 | 200 | GLOBAL | rail_glass | GameInit | GameInit__10rail_glassFv |
| 0x001f5780 | 92 | GLOBAL | rail_glass | GameDone | GameDone__10rail_glassFv |
| 0x001f57e0 | 216 | GLOBAL | rail_glass | AddWindow | AddWindow__10rail_glassFP11rail_window |
| 0x001f58c0 | 36 | 13 | rail_window | {ctor} | __ct__11rail_windowFv |
| 0x001f58f0 | 104 | GLOBAL | rail_glass | RestoreWindows | RestoreWindows__10rail_glassFv |
| 0x001f5960 | 160 | GLOBAL | rail_window | Restore | Restore__11rail_windowFP12I3D_material |
| 0x001f5a00 | 340 | GLOBAL | rail_window | Hit | Hit__11rail_windowFRC8S_vectorP12I3D_materialP12I3D_material |
| 0x001f5b60 | 68 | GLOBAL | C_railway | isInStation | isInStation__9C_railwayFv |
| 0x001f5bb0 | 108 | GLOBAL | C_railway | isInStop | isInStop__9C_railwayFv |
| 0x001f5c20 | 72 | GLOBAL | C_railway | GetVagon | GetVagon__9C_railwayFUi |
| 0x001f5c70 | 72 | GLOBAL | C_railway | MemWrite | MemWrite__9C_railwayFPPvPvUiUi |
| 0x001f5cc0 | 72 | GLOBAL | C_railway | MemRead | MemRead__9C_railwayFPPvPvUiUi |
| 0x001f5d10 | 68 | GLOBAL | C_railway | SaveGameGetSize | SaveGameGetSize__9C_railwayFv |
| 0x001f5d60 | 428 | GLOBAL | C_railway | SaveGameSave | SaveGameSave__9C_railwayFPPv |
| 0x001f5f10 | 1804 | GLOBAL | C_railway | SaveGameLoad | SaveGameLoad__9C_railwayFPPv |
| 0x001f6620 | 16 | GLOBAL | C_railway | Lock | Lock__9C_railwayFv |
| 0x001f6630 | 28 | GLOBAL | C_railway | Unlock | Unlock__9C_railwayFv |
| 0x001f6650 | 300 | GLOBAL | C_railway | IsOnSchedule | IsOnSchedule__9C_railwayFP10C_web_node |
| 0x001f6780 | 368 | GLOBAL | C_railway | DoPanic | DoPanic__9C_railwayFUiUi |
| 0x001f68f0 | 80 | GLOBAL | C_railway | SetRLWState | SetRLWState__9C_railwayFUi |
| 0x001f6940 | 820 | GLOBAL | C_railway | DoBrousicka | DoBrousicka__9C_railwayFv |
| 0x001f6c80 | 476 | GLOBAL | C_railway | ChangeState | ChangeState__9C_railwayFv |
| 0x001f6e60 | 976 | GLOBAL | C_railway | GetBoundPoints2D | GetBoundPoints2D__9C_railwayFRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame |
| 0x001f7230 | 252 | GLOBAL | C_railway | SetRainDummy | SetRainDummy__9C_railwayFbP9I3D_frame |
| 0x001f7330 | 828 | GLOBAL | C_railway | GeneratePersonTrafic | GeneratePersonTrafic__9C_railwayFi |
| 0x001f7670 | 432 | GLOBAL | C_railway | ReGenerate | ReGenerate__9C_railwayFP10C_web_node |
| 0x001f7820 | 196 | GLOBAL | C_railway | FreePersonTrafic | FreePersonTrafic__9C_railwayFv |
| 0x001f78f0 | 252 | GLOBAL | C_railway | ChangeSceneInit | ChangeSceneInit__9C_railwayFv |
| 0x001f79f0 | 360 | GLOBAL | C_railway | ChangeSceneDone | ChangeSceneDone__9C_railwayFv |
| 0x001f7b60 | 68 | GLOBAL | C_railway | GetCheckpoints | GetCheckpoints__9C_railwayFb |
| 0x001f7bb0 | 8 | 13 | C_railway | GetMaterialID | GetMaterialID__9C_railwayFP9I3D_frame |
| 0x001f7bc0 | 164 | 13 | C_railway | GetActorSpeed | GetActorSpeed__9C_railwayFR8S_vector |
| 0x001f7c70 | 132 | WEAK | std::__vector_imp<9raildummy,Q23std21allocator<9raildummy>,0> | erase | erase__Q23std56__vector_imp<9raildummy,Q23std21allocator<9raildummy>,0>FP9raildummy |
| 0x001f7d00 | 1180 | WEAK | std::__vector_imp<9raildummy,Q23std21allocator<9raildummy>,0> | insert | insert__Q23std56__vector_imp<9raildummy,Q23std21allocator<9raildummy>,0>FP9raildummyUiRC9raildummy |
| 0x001f81a0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std21allocator<9raildummy>,Ui,1> | swap | swap__Q310Metrowerks7details55compressed_pair_imp<Q23std21allocator<9raildummy>,Ui,1>FRQ310Metrowerks7details55compressed_pair_imp<Q23std21allocator<9raildummy>,Ui,1> |
| 0x001f81c0 | 1236 | WEAK | std::__vector_imp<6rentry,Q23std18allocator<6rentry>,0> | insert | insert__Q23std50__vector_imp<6rentry,Q23std18allocator<6rentry>,0>FP6rentryUiRC6rentry |
| 0x001f86a0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6rentry>,Ui,1> | swap | swap__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6rentry>,Ui,1>FRQ310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6rentry>,Ui,1> |
| 0x001f86c0 | 72 | WEAK | std::__vector_deleter<9raildummy,Q23std21allocator<9raildummy>> | clear | clear__Q23std58__vector_deleter<9raildummy,Q23std21allocator<9raildummy>>Fv |
| 0x001f8710 | 104 | WEAK | std::__vector_deleter<9raildummy,Q23std21allocator<9raildummy>> | {dtor} | __dt__Q23std58__vector_deleter<9raildummy,Q23std21allocator<9raildummy>>Fv |
| 0x001f8780 | 80 | WEAK | std::__vector_deleter<6rentry,Q23std18allocator<6rentry>> | clear | clear__Q23std52__vector_deleter<6rentry,Q23std18allocator<6rentry>>Fv |
| 0x001f87d0 | 104 | WEAK | std::__vector_deleter<6rentry,Q23std18allocator<6rentry>> | {dtor} | __dt__Q23std52__vector_deleter<6rentry,Q23std18allocator<6rentry>>Fv |
| 0x001f8840 | 140 | 13 | std::vector<P16S_rail_semaphore,Q23std30allocator<P16S_rail_semaphore>> | {dtor} | __dt__Q23std66vector<P16S_rail_semaphore,Q23std30allocator<P16S_rail_semaphore>>Fv |
| 0x001f88d0 | 140 | 13 | std::vector<P14S_shedule_item,Q23std28allocator<P14S_shedule_item>> | {dtor} | __dt__Q23std62vector<P14S_shedule_item,Q23std28allocator<P14S_shedule_item>>Fv |
| 0x001f8960 | 140 | 13 | std::vector<P17C_traffic_element,Q23std31allocator<P17C_traffic_element>> | {dtor} | __dt__Q23std68vector<P17C_traffic_element,Q23std31allocator<P17C_traffic_element>>Fv |
| 0x001f89f0 | 640 | GLOBAL | RMRF_S_SETREF | LoadRAWSet | LoadRAWSet__13RMRF_S_SETREFFPP13RMRF_S_SETREFPc |
| 0x001f8c70 | 24 | GLOBAL | C_rmorf | {ctor} | __ct__7C_rmorfFv |
| 0x001f8c90 | 176 | GLOBAL | C_rmorf | {dtor} | __dt__7C_rmorfFv |
| 0x001f8d40 | 192 | 13 | RMRF_S_SETREF | {dtor} | __dt__13RMRF_S_SETREFFv |
| 0x001f8e00 | 352 | GLOBAL | C_rmorf | Mission_Start | Mission_Start__7C_rmorfFv |
| 0x001f8f60 | 144 | 13 | fstring | {ctor} | __ct__7fstringFPCce |
| 0x001f8ff0 | 120 | GLOBAL | C_rmorf | Mission_Done | Mission_Done__7C_rmorfFv |
| 0x001f9070 | 2360 | GLOBAL | C_rmorf | Tick | Tick__7C_rmorfFi |
| 0x001f99b0 | 256 | GLOBAL | C_shvestky | {ctor} | __ct__10C_shvestkyFv |
| 0x001f9ab0 | 140 | 13 | std::vector<P5C_car,Q23std18allocator<P5C_car>> | {dtor} | __dt__Q23std42vector<P5C_car,Q23std18allocator<P5C_car>>Fv |
| 0x001f9b40 | 104 | 13 | std::__vector_imp<P5C_car,Q23std18allocator<P5C_car>,1> | {dtor} | __dt__Q23std50__vector_imp<P5C_car,Q23std18allocator<P5C_car>,1>Fv |
| 0x001f9bb0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q210C_shvestky18S_road_block_enemy>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q210C_shvestky18S_road_block_enemy>,Ui,1>FUi |
| 0x001f9bc0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std44allocator<Q210C_shvestky17S_road_block_iris>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q210C_shvestky17S_road_block_iris>,Ui,1>FUi |
| 0x001f9bd0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std43allocator<Q210C_shvestky16S_road_block_car>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details77compressed_pair_imp<Q23std43allocator<Q210C_shvestky16S_road_block_car>,Ui,1>FUi |
| 0x001f9be0 | 272 | GLOBAL | C_shvestky | {dtor} | __dt__10C_shvestkyFv |
| 0x001f9cf0 | 132 | GLOBAL | C_shvestky | GameInit | GameInit__10C_shvestkyFv |
| 0x001f9d80 | 1632 | GLOBAL | C_shvestky | GameDone | GameDone__10C_shvestkyFv |
| 0x001fa3e0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std43allocator<Q210C_shvestky16S_road_block_car>,Ui,1> | first | first__Q310Metrowerks7details77compressed_pair_imp<Q23std43allocator<Q210C_shvestky16S_road_block_car>,Ui,1>Fv |
| 0x001fa3f0 | 1240 | GLOBAL | C_shvestky | ChangeSceneInit | ChangeSceneInit__10C_shvestkyFv |
| 0x001fa8d0 | 8 | GLOBAL | C_shvestky | ChangeSceneDone | ChangeSceneDone__10C_shvestkyFv |
| 0x001fa8e0 | 92 | GLOBAL | C_shvestky | ChangeSceneVystrc | ChangeSceneVystrc__10C_shvestkyFv |
| 0x001fa940 | 136 | GLOBAL | C_shvestky | DezoSebralGumyIRadio | DezoSebralGumyIRadio__10C_shvestkyFP5C_car |
| 0x001fa9d0 | 96 | GLOBAL | C_shvestky | DezoVratilGumyAleRadioSiNechal | DezoVratilGumyAleRadioSiNechal__10C_shvestkyFP5C_car |
| 0x001faa30 | 3236 | GLOBAL | C_shvestky | Tick | Tick__10C_shvestkyFUi |
| 0x001fb6e0 | 2776 | 13 | C_shvestky | CreateShvestkyTest | CreateShvestkyTest__10C_shvestkyFv |
| 0x001fc1c0 | 2388 | 13 | C_shvestky | TestOverRed | TestOverRed__10C_shvestkyFv |
| 0x001fcb20 | 4388 | GLOBAL | C_shvestky | SetActive | SetActive__10C_shvestkyFbii |
| 0x001fdc50 | 416 | GLOBAL | C_shvestky | AddPerson | AddPerson__10C_shvestkyFP7C_human |
| 0x001fddf0 | 300 | GLOBAL | C_shvestky | DelPerson | DelPerson__10C_shvestkyFP7C_human |
| 0x001fdf20 | 232 | GLOBAL | C_shvestky | Firing | Firing__10C_shvestkyFP7C_human |
| 0x001fe010 | 264 | GLOBAL | C_shvestky | DriveCar | DriveCar__10C_shvestkyFP7C_humanP5C_car |
| 0x001fe120 | 1240 | GLOBAL | C_shvestky | CarCollision | CarCollision__10C_shvestkyFP5C_carPUif |
| 0x001fe600 | 136 | GLOBAL | C_shvestky | CarIllegalPath | CarIllegalPath__10C_shvestkyFP5C_car |
| 0x001fe690 | 184 | GLOBAL | C_shvestky | HumanBoxing | HumanBoxing__10C_shvestkyFP7C_human |
| 0x001fe750 | 168 | GLOBAL | C_shvestky | CarSteal | CarSteal__10C_shvestkyFP7C_humanbb |
| 0x001fe800 | 152 | GLOBAL | C_shvestky | ForceArrest | ForceArrest__10C_shvestkyFP7C_humanbb |
| 0x001fe8a0 | 224 | GLOBAL | C_shvestky | DestroySignalMark | DestroySignalMark__10C_shvestkyFP7C_human |
| 0x001fe980 | 104 | GLOBAL | C_shvestky | NextNaSrackySoudruh | NextNaSrackySoudruh__10C_shvestkyFP7C_human |
| 0x001fe9f0 | 216 | GLOBAL | C_shvestky | CorujuPoldumMasinu | CorujuPoldumMasinu__10C_shvestkyFP7C_humanP5C_car |
| 0x001fead0 | 120 | GLOBAL | C_shvestky | SerSeMuDoCesty | SerSeMuDoCesty__10C_shvestkyFP7C_human |
| 0x001feb50 | 84 | GLOBAL | C_shvestky | GetHumanStatus | GetHumanStatus__10C_shvestkyFP7C_human |
| 0x001febb0 | 76 | GLOBAL | C_shvestky | JakPicusNaTomJest | JakPicusNaTomJest__10C_shvestkyFP7C_human |
| 0x001fec00 | 76 | GLOBAL | C_shvestky | GetCrashTime | GetCrashTime__10C_shvestkyFP7C_human |
| 0x001fec50 | 76 | GLOBAL | C_shvestky | GetFireTime | GetFireTime__10C_shvestkyFP7C_human |
| 0x001feca0 | 76 | GLOBAL | C_shvestky | PocetNasranych | PocetNasranych__10C_shvestkyFP7C_human |
| 0x001fecf0 | 224 | GLOBAL | C_shvestky | PocetNasranychZivych | PocetNasranychZivych__10C_shvestkyFP7C_human |
| 0x001fedd0 | 76 | GLOBAL | C_shvestky | PocetPrestupku | PocetPrestupku__10C_shvestkyFP7C_human |
| 0x001fee20 | 108 | GLOBAL | C_shvestky | NeviDoKterePiceVlezl | NeviDoKterePiceVlezl__10C_shvestkyFP7C_human |
| 0x001fee90 | 76 | GLOBAL | C_shvestky | ChceHoOpichatKazdyPoldaNaOkresku | ChceHoOpichatKazdyPoldaNaOkresku__10C_shvestkyFP7C_human |
| 0x001feee0 | 464 | GLOBAL | C_shvestky | CreateCar | CreateCar__10C_shvestkyFR8S_vectorR8S_vectorb |
| 0x001ff0b0 | 528 | GLOBAL | C_shvestky | MasVolnouPlecku | MasVolnouPlecku__10C_shvestkyFPc |
| 0x001ff2c0 | 388 | GLOBAL | C_shvestky | GetTempCar | GetTempCar__10C_shvestkyFb |
| 0x001ff450 | 248 | GLOBAL | C_shvestky | FreeTempCar | FreeTempCar__10C_shvestkyFP5C_car |
| 0x001ff550 | 80 | GLOBAL | C_shvestky | GetFreeEnemy | GetFreeEnemy__10C_shvestkyFv |
| 0x001ff5a0 | 168 | GLOBAL | C_shvestky | FreeTempIris | FreeTempIris__10C_shvestkyFP8C_irenka |
| 0x001ff650 | 220 | GLOBAL | C_shvestky | GetTempEnemy | GetTempEnemy__10C_shvestkyFv |
| 0x001ff730 | 552 | GLOBAL | C_shvestky | FreeTempEnemy | FreeTempEnemy__10C_shvestkyFP8C_entity |
| 0x001ff960 | 8 | 13 | std::__vector_deleter<Q210C_shvestky18S_road_block_enemy,Q23std45allocator<Q210C_shvestky18S_road_block_enemy>> | data | data__Q23std106__vector_deleter<Q210C_shvestky18S_road_block_enemy,Q23std45allocator<Q210C_shvestky18S_road_block_enemy>>Fv |
| 0x001ff970 | 104 | GLOBAL | C_shvestky | FreeMyTempActor | FreeMyTempActor__10C_shvestkyFP7C_actor |
| 0x001ff9e0 | 136 | GLOBAL | C_shvestky | AddDoRiciActor | AddDoRiciActor__10C_shvestkyFP7C_actor |
| 0x001ffa70 | 208 | GLOBAL | C_shvestky | DelDoRiciActor | DelDoRiciActor__10C_shvestkyFP7C_actor |
| 0x001ffb40 | 168 | GLOBAL | C_shvestky | PocetPolicejnichPlechovekVRici | PocetPolicejnichPlechovekVRici__10C_shvestkyFv |
| 0x001ffbf0 | 1304 | GLOBAL | C_shvestky | TickDoRiciActors | TickDoRiciActors__10C_shvestkyFb |
| 0x00200110 | 296 | GLOBAL | C_shvestky | ZlikvidujVsechnyNasraneVyhozeneCuraky | ZlikvidujVsechnyNasraneVyhozeneCuraky__10C_shvestkyFv |
| 0x00200240 | 1968 | GLOBAL | C_shvestky | DalsiChalpiDoPartie | DalsiChalpiDoPartie__10C_shvestkyFP7C_humanUif |
| 0x002009f0 | 104 | GLOBAL | C_shvestky | GlobalDelNasrany | GlobalDelNasrany__10C_shvestkyFP7C_actor |
| 0x00200a60 | 136 | GLOBAL | C_shvestky::S_shvestky | AddNasrany | AddNasrany__Q210C_shvestky10S_shvestkyFP7C_actor |
| 0x00200af0 | 88 | GLOBAL | C_shvestky::S_shvestky | DelNasrany | DelNasrany__Q210C_shvestky10S_shvestkyFP7C_actor |
| 0x00200b50 | 192 | GLOBAL | C_shvestky | DamMuPokuritJesteVice | DamMuPokuritJesteVice__10C_shvestkyFP7C_humanb |
| 0x00200c10 | 472 | GLOBAL | C_shvestky | DobreMiPokuril_MuzeJitDom | DobreMiPokuril_MuzeJitDom__10C_shvestkyFP7C_humanb |
| 0x00200df0 | 2052 | GLOBAL | C_shvestky | VasekSelChlastatNaOslavu | VasekSelChlastatNaOslavu__10C_shvestkyFP7C_humanRC8S_vectorfbb |
| 0x00201600 | 268 | GLOBAL | C_shvestky | NabonzovalJsemHo | NabonzovalJsemHo__10C_shvestkyFP7C_human |
| 0x00201710 | 136 | GLOBAL | C_shvestky | DalJsemPochuzkariDoDrzky | DalJsemPochuzkariDoDrzky__10C_shvestkyFP7C_human |
| 0x002017a0 | 248 | GLOBAL | C_shvestky | StrcilJsemHoPoldumDoVyfuku | StrcilJsemHoPoldumDoVyfuku__10C_shvestkyFP7C_actorP5C_carf |
| 0x002018a0 | 120 | GLOBAL | C_shvestky | ZalozSpisATahniDoRiti_Haleluja | ZalozSpisATahniDoRiti_Haleluja__10C_shvestkyFP7C_humanP7C_actorb |
| 0x00201920 | 200 | GLOBAL | C_shvestky | IdePoliciePoPlayeroviBoPumper | IdePoliciePoPlayeroviBoPumper__10C_shvestkyFv |
| 0x002019f0 | 168 | GLOBAL | C_shvestky | RenamePoliceCar | RenamePoliceCar__10C_shvestkyFP5C_car |
| 0x00201aa0 | 696 | GLOBAL | C_shvestky | GetPolicePosForKokot | GetPolicePosForKokot__10C_shvestkyFP7C_humanR18S_police_interface |
| 0x00201d60 | 1600 | GLOBAL | C_shvestky | BugZamrdanychTesteruZTepleDve_410 | BugZamrdanychTesteruZTepleDve_410__10C_shvestkyFP7C_humaniR18S_police_interface |
| 0x002023a0 | 100 | GLOBAL | C_shvestky | JeToDutolebecniAuto | JeToDutolebecniAuto__10C_shvestkyFP5C_carb |
| 0x00202410 | 72 | WEAK | std::__vector_deleter<Q210C_shvestky16S_road_block_car,Q23std43allocator<Q210C_shvestky16S_road_block_car>> | clear | clear__Q23std102__vector_deleter<Q210C_shvestky16S_road_block_car,Q23std43allocator<Q210C_shvestky16S_road_block_car>>Fv |
| 0x00202460 | 1068 | WEAK | std::__vector_imp<Q210C_shvestky16S_road_block_car,Q23std43allocator<Q210C_shvestky16S_road_block_car>,0> | insert | insert__Q23std100__vector_imp<Q210C_shvestky16S_road_block_car,Q23std43allocator<Q210C_shvestky16S_road_block_car>,0>FPQ210C_shvestky16S_road_block_carUiRCQ210C_shvestky16S_road_block_car |
| 0x00202890 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std43allocator<Q210C_shvestky16S_road_block_car>,Ui,1> | swap | swap__Q310Metrowerks7details77compressed_pair_imp<Q23std43allocator<Q210C_shvestky16S_road_block_car>,Ui,1>FRQ310Metrowerks7details77compressed_pair_imp<Q23std43allocator<Q210C_shvestky16S_road_block_car>,Ui,1> |
| 0x002028b0 | 1052 | WEAK | std::__vector_imp<Q210C_shvestky18S_road_block_enemy,Q23std45allocator<Q210C_shvestky18S_road_block_enemy>,0> | insert | insert__Q23std104__vector_imp<Q210C_shvestky18S_road_block_enemy,Q23std45allocator<Q210C_shvestky18S_road_block_enemy>,0>FPQ210C_shvestky18S_road_block_enemyUiRCQ210C_shvestky18S_road_block_enemy |
| 0x00202cd0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q210C_shvestky18S_road_block_enemy>,Ui,1> | swap | swap__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q210C_shvestky18S_road_block_enemy>,Ui,1>FRQ310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q210C_shvestky18S_road_block_enemy>,Ui,1> |
| 0x00202cf0 | 104 | WEAK | std::__vector_deleter<Q210C_shvestky16S_road_block_car,Q23std43allocator<Q210C_shvestky16S_road_block_car>> | {dtor} | __dt__Q23std102__vector_deleter<Q210C_shvestky16S_road_block_car,Q23std43allocator<Q210C_shvestky16S_road_block_car>>Fv |
| 0x00202d60 | 72 | WEAK | std::__vector_deleter<Q210C_shvestky17S_road_block_iris,Q23std44allocator<Q210C_shvestky17S_road_block_iris>> | clear | clear__Q23std104__vector_deleter<Q210C_shvestky17S_road_block_iris,Q23std44allocator<Q210C_shvestky17S_road_block_iris>>Fv |
| 0x00202db0 | 104 | WEAK | std::__vector_deleter<Q210C_shvestky17S_road_block_iris,Q23std44allocator<Q210C_shvestky17S_road_block_iris>> | {dtor} | __dt__Q23std104__vector_deleter<Q210C_shvestky17S_road_block_iris,Q23std44allocator<Q210C_shvestky17S_road_block_iris>>Fv |
| 0x00202e20 | 72 | WEAK | std::__vector_deleter<Q210C_shvestky18S_road_block_enemy,Q23std45allocator<Q210C_shvestky18S_road_block_enemy>> | clear | clear__Q23std106__vector_deleter<Q210C_shvestky18S_road_block_enemy,Q23std45allocator<Q210C_shvestky18S_road_block_enemy>>Fv |
| 0x00202e70 | 104 | WEAK | std::__vector_deleter<Q210C_shvestky18S_road_block_enemy,Q23std45allocator<Q210C_shvestky18S_road_block_enemy>> | {dtor} | __dt__Q23std106__vector_deleter<Q210C_shvestky18S_road_block_enemy,Q23std45allocator<Q210C_shvestky18S_road_block_enemy>>Fv |
| 0x00202ee0 | 68 | GLOBAL | C_sledovacka | {ctor} | __ct__12C_sledovackaFv |
| 0x00202f30 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std36allocator<Q212C_sledovacka8S_BRIDGE>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details70compressed_pair_imp<Q23std36allocator<Q212C_sledovacka8S_BRIDGE>,Ui,1>FUi |
| 0x00202f40 | 144 | GLOBAL | C_sledovacka | {dtor} | __dt__12C_sledovackaFv |
| 0x00202fd0 | 8 | GLOBAL | C_sledovacka | EnableChangeTables | EnableChangeTables__12C_sledovackaFb |
| 0x00202fe0 | 588 | GLOBAL | C_sledovacka | Init | Init__12C_sledovackaFv |
| 0x00203230 | 244 | GLOBAL | C_sledovacka | Done | Done__12C_sledovackaFv |
| 0x00203330 | 116 | GLOBAL | C_sledovacka | HoniMiHoNekdoZKanclu | HoniMiHoNekdoZKanclu__12C_sledovackaFP5C_car |
| 0x002033b0 | 1664 | GLOBAL | C_sledovacka | AddCarHunt | AddCarHunt__12C_sledovackaFP5C_carP7C_actorUiUiRC8S_vectorf |
| 0x00203a30 | 128 | 13 | C_sledovacka::S_ELEMENT | {ctor} | __ct__Q212C_sledovacka9S_ELEMENTFRCQ212C_sledovacka9S_ELEMENT |
| 0x00203ab0 | 348 | 13 | gif::vector<Q212C_sledovacka9S_ELEMENT,8> | reserve | reserve__Q23gif36vector<Q212C_sledovacka9S_ELEMENT,8>Fi |
| 0x00203c10 | 3824 | GLOBAL | C_sledovacka | AddCarEscape | AddCarEscape__12C_sledovackaFP5C_carP7C_actorRC8S_vectorbfff |
| 0x00204b00 | 5420 | GLOBAL | C_sledovacka | AddCarMoveTo | AddCarMoveTo__12C_sledovackaFP5C_carRC8S_vectorbbUif |
| 0x00206030 | 572 | GLOBAL | C_sledovacka | RemoveCar | RemoveCar__12C_sledovackaFP5C_car |
| 0x00206270 | 128 | 13 | C_sledovacka::S_ELEMENT | operator= | __as__Q212C_sledovacka9S_ELEMENTFRCQ212C_sledovacka9S_ELEMENT |
| 0x002062f0 | 4120 | GLOBAL | C_sledovacka | RecompilePath | RecompilePath__12C_sledovackaFi |
| 0x00207310 | 668 | GLOBAL | C_sledovacka | PreparePath | PreparePath__12C_sledovackaFi |
| 0x002075b0 | 8164 | GLOBAL | C_sledovacka | TickDriver | TickDriver__12C_sledovackaFiUi |
| 0x002095a0 | 26440 | GLOBAL | C_sledovacka | Tick | Tick__12C_sledovackaFUi |
| 0x0020fcf0 | 140 | GLOBAL | C_sledovacka | ActionStatus | ActionStatus__12C_sledovackaFP5C_car |
| 0x0020fd80 | 140 | GLOBAL | C_sledovacka | SetActionStatus | SetActionStatus__12C_sledovackaFP5C_car8E_STATUS |
| 0x0020fe10 | 140 | GLOBAL | C_sledovacka | GetVoleMode | GetVoleMode__12C_sledovackaFP5C_car |
| 0x0020fea0 | 516 | GLOBAL | C_sledovacka | VypicujMiToyotu | VypicujMiToyotu__12C_sledovackaFP7C_actor |
| 0x002100b0 | 184 | GLOBAL | C_sledovacka | DevaaateeeZasNeznaCas | DevaaateeeZasNeznaCas__12C_sledovackaFP5C_car |
| 0x00210170 | 400 | GLOBAL | C_sledovacka | SaveGameGetSize | SaveGameGetSize__12C_sledovackaFv |
| 0x00210300 | 1916 | GLOBAL | C_sledovacka | SaveGameSave | SaveGameSave__12C_sledovackaFPPv |
| 0x00210a80 | 2804 | GLOBAL | C_sledovacka | SaveGameLoad | SaveGameLoad__12C_sledovackaFPPv |
| 0x00211580 | 872 | 13 | gif::vector<10S_car_node,8> | push_back | push_back__Q23gif22vector<10S_car_node,8>FRC10S_car_node |
| 0x002118f0 | 148 | WEAK | std::__vector_pod<Us,Q23std13allocator<Us>> | erase | erase__Q23std38__vector_pod<Us,Q23std13allocator<Us>>FPUs |
| 0x00211990 | 1052 | WEAK | std::__vector_imp<Q212C_sledovacka8S_BRIDGE,Q23std36allocator<Q212C_sledovacka8S_BRIDGE>,0> | insert | insert__Q23std86__vector_imp<Q212C_sledovacka8S_BRIDGE,Q23std36allocator<Q212C_sledovacka8S_BRIDGE>,0>FPQ212C_sledovacka8S_BRIDGEUiRCQ212C_sledovacka8S_BRIDGE |
| 0x00211db0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std36allocator<Q212C_sledovacka8S_BRIDGE>,Ui,1> | swap | swap__Q310Metrowerks7details70compressed_pair_imp<Q23std36allocator<Q212C_sledovacka8S_BRIDGE>,Ui,1>FRQ310Metrowerks7details70compressed_pair_imp<Q23std36allocator<Q212C_sledovacka8S_BRIDGE>,Ui,1> |
| 0x00211dd0 | 72 | WEAK | std::__vector_deleter<Q212C_sledovacka8S_BRIDGE,Q23std36allocator<Q212C_sledovacka8S_BRIDGE>> | clear | clear__Q23std88__vector_deleter<Q212C_sledovacka8S_BRIDGE,Q23std36allocator<Q212C_sledovacka8S_BRIDGE>>Fv |
| 0x00211e20 | 104 | WEAK | std::__vector_deleter<Q212C_sledovacka8S_BRIDGE,Q23std36allocator<Q212C_sledovacka8S_BRIDGE>> | {dtor} | __dt__Q23std88__vector_deleter<Q212C_sledovacka8S_BRIDGE,Q23std36allocator<Q212C_sledovacka8S_BRIDGE>>Fv |
| 0x00211e90 | 68 | GLOBAL | C_smoke_trail | {ctor} | __ct__13C_smoke_trailFv |
| 0x00211ee0 | 96 | GLOBAL | C_smoke_trail | {dtor} | __dt__13C_smoke_trailFv |
| 0x00211f40 | 44 | GLOBAL | C_smoke_trail | Init | Init__13C_smoke_trailFP9I3D_frame |
| 0x00211f70 | 160 | GLOBAL | C_smoke_trail | AI | AI__13C_smoke_trailFUi |
| 0x00212010 | 68 | GLOBAL | C_smoke_trail | Update | Update__13C_smoke_trailFUi |
| 0x00212060 | 8 | GLOBAL | C_smoke_trail | NetDirect | NetDirect__13C_smoke_trailFUi |
| 0x00212070 | 84 | GLOBAL | C_smoke_trail | GameInit | GameInit__13C_smoke_trailFv |
| 0x002120d0 | 108 | GLOBAL | C_smoke_trail | GameDone | GameDone__13C_smoke_trailFv |
| 0x00212140 | 8 | GLOBAL | C_smoke_trail | SaveGameGetSize | SaveGameGetSize__13C_smoke_trailFv |
| 0x00212150 | 8 | GLOBAL | C_smoke_trail | SaveGameSave | SaveGameSave__13C_smoke_trailFPPv |
| 0x00212160 | 8 | GLOBAL | C_smoke_trail | SaveGameLoad | SaveGameLoad__13C_smoke_trailFPPv |
| 0x00212170 | 8 | GLOBAL | C_smoke_trail | Collision | Collision__13C_smoke_trailFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00212180 | 8 | GLOBAL | C_smoke_trail | Hit | Hit__13C_smoke_trailF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00212190 | 8 | 13 | C_smoke_trail | SaveData | SaveData__13C_smoke_trailFP7C_chunk |
| 0x002121a0 | 8 | 13 | C_smoke_trail | LoadData | LoadData__13C_smoke_trailFP7C_chunk |
| 0x002121b0 | 116 | GLOBAL | C_traffic_car | {ctor} | __ct__13C_traffic_carFv |
| 0x00212230 | 112 | GLOBAL | C_traffic_car | {dtor} | __dt__13C_traffic_carFv |
| 0x002122a0 | 84 | GLOBAL | C_traffic_car | Init | Init__13C_traffic_carFP9I3D_frame |
| 0x00212300 | 520 | GLOBAL | C_traffic_car | ChangeState | ChangeState__13C_traffic_carFv |
| 0x00212510 | 496 | GLOBAL | C_traffic_car | Scipni | Scipni__13C_traffic_carFb |
| 0x00212700 | 196 | GLOBAL | C_traffic_car | DuplicateFrom | DuplicateFrom__13C_traffic_carFP7C_actor |
| 0x002127d0 | 8 | GLOBAL | C_traffic_car | Death | Death__13C_traffic_carFv |
| 0x002127e0 | 384 | GLOBAL | C_traffic_car | SaveData | SaveData__13C_traffic_carFP7C_chunk |
| 0x00212960 | 1016 | GLOBAL | C_traffic_car | LoadData | LoadData__13C_traffic_carFP7C_chunk |
| 0x00212d60 | 8 | GLOBAL | C_traffic_car | Collision | Collision__13C_traffic_carFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00212d70 | 8 | GLOBAL | C_traffic_car | NetDirect | NetDirect__13C_traffic_carFUi |
| 0x00212d80 | 2192 | GLOBAL | C_traffic_car | DeactivateElem | DeactivateElem__13C_traffic_carFib |
| 0x00213610 | 14300 | GLOBAL | C_traffic_car | ActivateElems | ActivateElems__13C_traffic_carFbi |
| 0x00216df0 | 36 | 13 |  | lsi_fabs__Ff | lsi_fabs__Ff |
| 0x00216e20 | 4616 | 13 | C_traffic_car | PrepareNextDir | PrepareNextDir__13C_traffic_carFib |
| 0x00218030 | 1224 | GLOBAL | C_traffic_car | GetNewPosInfo | GetNewPosInfo__13C_traffic_carFRUsRUs |
| 0x00218500 | 1928 | 13 | C_traffic_car | FindLastCrossing | FindLastCrossing__13C_traffic_carFRUsRi |
| 0x00218c90 | 1624 | 13 | C_traffic_car | FindNodesIndex | FindNodesIndex__13C_traffic_carFRUsRUsRi |
| 0x002192f0 | 264 | 13 | C_traffic_car | ClearBlocking | ClearBlocking__13C_traffic_carFii |
| 0x00219400 | 22344 | GLOBAL | C_traffic_car | AI | AI__13C_traffic_carFUi |
| 0x0021eb50 | 8 | 13 | std::__vector_deleter<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>> | data | data__Q23std64__vector_deleter<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>>Fv |
| 0x0021eb60 | 9400 | 13 | C_traffic_car | LockCrossing | LockCrossing__13C_traffic_carFii |
| 0x00221020 | 840 | GLOBAL | C_traffic_car | Update | Update__13C_traffic_carFUi |
| 0x00221370 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_MAFIAN>,Ui,1> | first | first__Q310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_MAFIAN>,Ui,1>Fv |
| 0x00221380 | 15412 | GLOBAL | C_traffic_car | GameInit | GameInit__13C_traffic_carFv |
| 0x00224fc0 | 1448 | GLOBAL | C_traffic_car | GameDone | GameDone__13C_traffic_carFv |
| 0x00225570 | 332 | GLOBAL | C_traffic_car | Corovacka | Corovacka__13C_traffic_carFP5C_car |
| 0x002256c0 | 12 | GLOBAL | C_traffic_car | GetParkingCars | GetParkingCars__13C_traffic_carFv |
| 0x002256d0 | 1364 | GLOBAL | C_traffic_car | ChodDoRici | ChodDoRici__13C_traffic_carFP9I3D_modelP5C_car |
| 0x00225c30 | 3912 | GLOBAL | C_traffic_car | HulaaaaaMuf | HulaaaaaMuf__13C_traffic_carFP7C_actorRQ23std42vector<P5C_car,Q23std18allocator<P5C_car>>ibP5C_carf |
| 0x00226b80 | 16 | 13 | std::__vector_imp<PQ213C_traffic_car9S_ELEMENT,Q23std39allocator<PQ213C_traffic_car9S_ELEMENT>,1> | operator[] | __vc__Q23std92__vector_imp<PQ213C_traffic_car9S_ELEMENT,Q23std39allocator<PQ213C_traffic_car9S_ELEMENT>,1>FUi |
| 0x00226b90 | 140 | GLOBAL | C_traffic_car | CoDelasTyCuraku | CoDelasTyCuraku__13C_traffic_carFP5C_carP7C_actorf |
| 0x00226c20 | 212 | GLOBAL | C_traffic_car | CoDelasTyCuraku1 | CoDelasTyCuraku1__13C_traffic_carFP5C_carP7C_actorf |
| 0x00226d00 | 36 | GLOBAL | C_traffic_car | ChlupatejVystrik | ChlupatejVystrik__13C_traffic_carFv |
| 0x00226d30 | 36 | GLOBAL | C_traffic_car | MafianVystrik | MafianVystrik__13C_traffic_carFv |
| 0x00226d60 | 4296 | GLOBAL | C_traffic_car | Krles | Krles__13C_traffic_carFP7C_actorP7C_actor |
| 0x00227e30 | 2652 | GLOBAL | C_traffic_car | SebraliMiKaaaruSvine | SebraliMiKaaaruSvine__13C_traffic_carFP5C_car |
| 0x00228890 | 8 | GLOBAL | C_traffic_car | DejSiZatimSlofiiika | DejSiZatimSlofiiika__13C_traffic_carFb |
| 0x002288a0 | 256 | GLOBAL | C_traffic_car | GetBridgeCarCount | GetBridgeCarCount__13C_traffic_carFP7C_actor |
| 0x002289a0 | 376 | GLOBAL | C_traffic_car | IsPossibleColl | IsPossibleColl__13C_traffic_carFP7C_actorR8S_vectorR8S_vectorfRfPP7C_actor |
| 0x00228b20 | 812 | GLOBAL | C_traffic_car | Hit | Hit__13C_traffic_carF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00228e50 | 144 | GLOBAL | C_traffic_car | ChcesMeSejmout | ChcesMeSejmout__13C_traffic_carFRC8S_vector |
| 0x00228ee0 | 256 | GLOBAL | C_traffic_car | TakATedSiVasKupim_Vsecky | TakATedSiVasKupim_Vsecky__13C_traffic_carFb |
| 0x00228fe0 | 636 | GLOBAL | C_traffic_car | DostalJsemRanuKachlou | DostalJsemRanuKachlou__13C_traffic_carFP5C_carP9I3D_frame |
| 0x00229260 | 4312 | GLOBAL | C_traffic_car | NoTakMiTedaDajTuPleckuJaJuOdvezu | NoTakMiTedaDajTuPleckuJaJuOdvezu__13C_traffic_carFP5C_car |
| 0x0022a340 | 1064 | GLOBAL | C_traffic_car | VypraskeeeKaaaru | VypraskeeeKaaaru__13C_traffic_carFP7C_actor |
| 0x0022a770 | 144 | GLOBAL | C_traffic_car | ChangeSceneInit | ChangeSceneInit__13C_traffic_carFv |
| 0x0022a800 | 216 | GLOBAL | C_traffic_car | ChangeSceneDone | ChangeSceneDone__13C_traffic_carFv |
| 0x0022a8e0 | 548 | GLOBAL | C_traffic_car | InvalidateActor | InvalidateActor__13C_traffic_carFP7C_actor |
| 0x0022ab10 | 32 | GLOBAL | C_traffic_car | SaveGameGetSize | SaveGameGetSize__13C_traffic_carFv |
| 0x0022ab30 | 76 | GLOBAL | C_traffic_car | SaveGameSave | SaveGameSave__13C_traffic_carFPPv |
| 0x0022ab80 | 84 | GLOBAL | C_traffic_car | SaveGameLoad | SaveGameLoad__13C_traffic_carFPPv |
| 0x0022abe0 | 80 | WEAK | std::__vector_deleter<Q213C_traffic_car8S_MAFIAN,Q23std37allocator<Q213C_traffic_car8S_MAFIAN>> | clear | clear__Q23std90__vector_deleter<Q213C_traffic_car8S_MAFIAN,Q23std37allocator<Q213C_traffic_car8S_MAFIAN>>Fv |
| 0x0022ac30 | 284 | WEAK | std::__vector_imp<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>,0> | erase | erase__Q23std62__vector_imp<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>,0>FP11S_LOCKCROSS |
| 0x0022ad50 | 1564 | WEAK | std::__vector_imp<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>,0> | insert | insert__Q23std62__vector_imp<11S_LOCKCROSS,Q23std24allocator<11S_LOCKCROSS>,0>FP11S_LOCKCROSSUiRC11S_LOCKCROSS |
| 0x0022b370 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std24allocator<11S_LOCKCROSS>,Ui,1> | swap | swap__Q310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_LOCKCROSS>,Ui,1>FRQ310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_LOCKCROSS>,Ui,1> |
| 0x0022b390 | 1052 | WEAK | std::__vector_imp<Q213C_traffic_car8S_BRIDGE,Q23std37allocator<Q213C_traffic_car8S_BRIDGE>,0> | insert | insert__Q23std88__vector_imp<Q213C_traffic_car8S_BRIDGE,Q23std37allocator<Q213C_traffic_car8S_BRIDGE>,0>FPQ213C_traffic_car8S_BRIDGEUiRCQ213C_traffic_car8S_BRIDGE |
| 0x0022b7b0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_BRIDGE>,Ui,1> | swap | swap__Q310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_BRIDGE>,Ui,1>FRQ310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_BRIDGE>,Ui,1> |
| 0x0022b7d0 | 1172 | WEAK | std::__vector_imp<Q213C_traffic_car8S_MAFIAN,Q23std37allocator<Q213C_traffic_car8S_MAFIAN>,0> | insert | insert__Q23std88__vector_imp<Q213C_traffic_car8S_MAFIAN,Q23std37allocator<Q213C_traffic_car8S_MAFIAN>,0>FPQ213C_traffic_car8S_MAFIANUiRCQ213C_traffic_car8S_MAFIAN |
| 0x0022bc70 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_MAFIAN>,Ui,1> | swap | swap__Q310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_MAFIAN>,Ui,1>FRQ310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_MAFIAN>,Ui,1> |
| 0x0022bc90 | 72 | WEAK | std::__vector_deleter<Q213C_traffic_car8S_BRIDGE,Q23std37allocator<Q213C_traffic_car8S_BRIDGE>> | clear | clear__Q23std90__vector_deleter<Q213C_traffic_car8S_BRIDGE,Q23std37allocator<Q213C_traffic_car8S_BRIDGE>>Fv |
| 0x0022bce0 | 104 | WEAK | std::__vector_deleter<Q213C_traffic_car8S_BRIDGE,Q23std37allocator<Q213C_traffic_car8S_BRIDGE>> | {dtor} | __dt__Q23std90__vector_deleter<Q213C_traffic_car8S_BRIDGE,Q23std37allocator<Q213C_traffic_car8S_BRIDGE>>Fv |
| 0x0022bd50 | 104 | WEAK | std::__vector_deleter<Q213C_traffic_car8S_MAFIAN,Q23std37allocator<Q213C_traffic_car8S_MAFIAN>> | {dtor} | __dt__Q23std90__vector_deleter<Q213C_traffic_car8S_MAFIAN,Q23std37allocator<Q213C_traffic_car8S_MAFIAN>>Fv |
| 0x0022bdc0 | 92 | 13 | std::vector<Q213C_traffic_car8S_MAFIAN,Q23std37allocator<Q213C_traffic_car8S_MAFIAN>> | {dtor} | __dt__Q23std80vector<Q213C_traffic_car8S_MAFIAN,Q23std37allocator<Q213C_traffic_car8S_MAFIAN>>Fv |
| 0x0022be20 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_MAFIAN>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_MAFIAN>,Ui,1>FUi |
| 0x0022be30 | 92 | 13 | std::vector<Q213C_traffic_car8S_BRIDGE,Q23std37allocator<Q213C_traffic_car8S_BRIDGE>> | {dtor} | __dt__Q23std80vector<Q213C_traffic_car8S_BRIDGE,Q23std37allocator<Q213C_traffic_car8S_BRIDGE>>Fv |
| 0x0022be90 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_BRIDGE>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q213C_traffic_car8S_BRIDGE>,Ui,1>FUi |
| 0x0022bea0 | 140 | 13 | std::vector<PQ213C_traffic_car9S_ELEMENT,Q23std39allocator<PQ213C_traffic_car9S_ELEMENT>> | {dtor} | __dt__Q23std84vector<PQ213C_traffic_car9S_ELEMENT,Q23std39allocator<PQ213C_traffic_car9S_ELEMENT>>Fv |
| 0x0022bf30 | 304 | GLOBAL | C_turnout | {ctor} | __ct__9C_turnoutFv |
| 0x0022c060 | 96 | GLOBAL | C_turnout | {dtor} | __dt__9C_turnoutFv |
| 0x0022c0c0 | 44 | GLOBAL | C_turnout | Init | Init__9C_turnoutFP9I3D_frame |
| 0x0022c0f0 | 448 | GLOBAL | C_turnout | SaveData | SaveData__9C_turnoutFP7C_chunk |
| 0x0022c2b0 | 1204 | GLOBAL | C_turnout | LoadData | LoadData__9C_turnoutFP7C_chunk |
| 0x0022c770 | 132 | GLOBAL | C_turnout | DuplicateFrom | DuplicateFrom__9C_turnoutFP7C_actor |
| 0x0022c800 | 1044 | GLOBAL | C_turnout | GameInit | GameInit__9C_turnoutFv |
| 0x0022cc20 | 248 | GLOBAL | C_turnout | GameDone | GameDone__9C_turnoutFv |
| 0x0022cd20 | 700 | GLOBAL | C_turnout | AI | AI__9C_turnoutFUi |
| 0x0022cfe0 | 272 | GLOBAL | C_turnout | Update | Update__9C_turnoutFUi |
| 0x0022d0f0 | 176 | GLOBAL | C_turnout | Start | Start__9C_turnoutFv |
| 0x0022d1a0 | 8 | GLOBAL | C_turnout | Hit | Hit__9C_turnoutF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x0022d1b0 | 216 | GLOBAL | C_turnout | SetCollPos | SetCollPos__9C_turnoutFv |
| 0x0022d290 | 208 | GLOBAL | C_turnout | NodesOK | NodesOK__9C_turnoutFRUiRUiRUi |
| 0x0022d360 | 32 | GLOBAL | C_turnout | SaveGameGetSize | SaveGameGetSize__9C_turnoutFv |
| 0x0022d380 | 96 | GLOBAL | C_turnout | SaveGameSave | SaveGameSave__9C_turnoutFPPv |
| 0x0022d3e0 | 640 | GLOBAL | C_turnout | SaveGameLoad | SaveGameLoad__9C_turnoutFPPv |
| 0x0022d660 | 1060 | GLOBAL | C_Vehicle | {ctor} | __ct__9C_VehicleFv |
| 0x0022da90 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std32allocator<19S_DYN_COLL_SETTINGS>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details66compressed_pair_imp<Q23std32allocator<19S_DYN_COLL_SETTINGS>,Ui,1>FUi |
| 0x0022daa0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18S_DYN_COLL_MOVABLE>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_DYN_COLL_MOVABLE>,Ui,1>FUi |
| 0x0022dab0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std29allocator<16S_COLBODY_EMITOR>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details63compressed_pair_imp<Q23std29allocator<16S_COLBODY_EMITOR>,Ui,1>FUi |
| 0x0022dac0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_DEFORM_ZONE>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_DEFORM_ZONE>,Ui,1>FUi |
| 0x0022dad0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std25allocator<12S_LPROJECTOR>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details59compressed_pair_imp<Q23std25allocator<12S_LPROJECTOR>,Ui,1>FUi |
| 0x0022dae0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std19allocator<7S_STRIP>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details53compressed_pair_imp<Q23std19allocator<7S_STRIP>,Ui,1>FUi |
| 0x0022daf0 | 13560 | GLOBAL | C_Vehicle | Init | Init__9C_VehicleFP9I3D_frame |
| 0x00230ff0 | 104 | 13 | std::__vector_imp<f,Q23std12allocator<f>,1> | {dtor} | __dt__Q23std38__vector_imp<f,Q23std12allocator<f>,1>Fv |
| 0x00231060 | 364 | 13 | gif::vector<Q29C_Vehicle6S_SEAT,8> | reserve | reserve__Q23gif29vector<Q29C_Vehicle6S_SEAT,8>Fi |
| 0x002311d0 | 348 | 13 | gif::vector<Q29C_Vehicle9S_BALLAST,8> | reserve | reserve__Q23gif32vector<Q29C_Vehicle9S_BALLAST,8>Fi |
| 0x00231330 | 152 | 13 | C_Vehicle::S_DOOR | {ctor} | __ct__Q29C_Vehicle6S_DOORFRCQ29C_Vehicle6S_DOOR |
| 0x002313d0 | 276 | 13 | gif::vector<Q29C_Vehicle6S_DOOR,8> | reserve | reserve__Q23gif29vector<Q29C_Vehicle6S_DOOR,8>Fi |
| 0x002314f0 | 348 | 13 | gif::vector<9S_EXHAUST,8> | reserve | reserve__Q23gif20vector<9S_EXHAUST,8>Fi |
| 0x00231650 | 2088 | GLOBAL | C_Vehicle | InitAxis | InitAxis__9C_VehicleFv |
| 0x00231e80 | 64 | 13 | S_AXLE | {ctor} | __ct__6S_AXLEFRC6S_AXLE |
| 0x00231ec0 | 308 | 13 | gif::vector<6S_AXLE,8> | reserve | reserve__Q23gif17vector<6S_AXLE,8>Fi |
| 0x00232000 | 4132 | GLOBAL | C_Vehicle | InitLights | InitLights__9C_VehicleFv |
| 0x00233030 | 104 | 13 | S_LIGHT | {ctor} | __ct__7S_LIGHTFRC7S_LIGHT |
| 0x002330a0 | 276 | 13 | gif::vector<7S_LIGHT,8> | reserve | reserve__Q23gif18vector<7S_LIGHT,8>Fi |
| 0x002331c0 | 988 | GLOBAL | C_Vehicle | InitSpecialLightProjectors | InitSpecialLightProjectors__9C_VehicleFv |
| 0x002335a0 | 520 | GLOBAL | C_Vehicle | CloseSpecialLightProjectors | CloseSpecialLightProjectors__9C_VehicleFv |
| 0x002337b0 | 712 | GLOBAL | C_Vehicle | {dtor} | __dt__9C_VehicleFv |
| 0x00233a80 | 1676 | GLOBAL | C_Vehicle | CloseVehicle | CloseVehicle__9C_VehicleFv |
| 0x00234110 | 144 | GLOBAL | C_Vehicle | ReleaseBodyColls | ReleaseBodyColls__9C_VehicleFP14S_COLBODY_HEAD |
| 0x002341a0 | 8 | GLOBAL | C_Vehicle | CreateLightProjector | CreateLightProjector__9C_VehicleFP9I3D_framePP9I3D_light |
| 0x002341b0 | 1712 | GLOBAL | C_Vehicle | CreateSpecialLightProjector | CreateSpecialLightProjector__9C_VehicleFP9I3D_framePP9I3D_light |
| 0x00234860 | 1080 | GLOBAL | C_Vehicle | Deactivate | Deactivate__9C_VehicleFUi |
| 0x00234ca0 | 1324 | GLOBAL | C_Vehicle | Reset1 | Reset1__9C_VehicleFv |
| 0x002351d0 | 2460 | GLOBAL | C_Vehicle | Reset | Reset__9C_VehicleFfb |
| 0x00235b70 | 9872 | GLOBAL | C_Vehicle | GameInit | GameInit__9C_VehicleFv |
| 0x00238200 | 112 | 13 | tDynamicCollObject | {ctor} | __ct__18tDynamicCollObjectFRC18tDynamicCollObject |
| 0x00238270 | 348 | 13 | gif::vector<18tDynamicCollObject,8> | reserve | reserve__Q23gif30vector<18tDynamicCollObject,8>Fi |
| 0x002383d0 | 1096 | GLOBAL | C_Vehicle | GameDone | GameDone__9C_VehicleFv |
| 0x00238820 | 484 | GLOBAL | C_Vehicle | SetTyres | SetTyres__9C_VehicleFPCQ23std42vector<6S_TYRE,Q23std18allocator<6S_TYRE>> |
| 0x00238a10 | 144 | GLOBAL | C_Vehicle | CloseTyres | CloseTyres__9C_VehicleFv |
| 0x00238aa0 | 244 | GLOBAL | C_Vehicle | SetVehicleMaterials | SetVehicleMaterials__9C_VehicleFPCQ23std60vector<14S_VEH_MATERIAL,Q23std27allocator<14S_VEH_MATERIAL>> |
| 0x00238ba0 | 12 | GLOBAL | C_Vehicle | CloseVehicleMaterials | CloseVehicleMaterials__9C_VehicleFv |
| 0x00238bb0 | 432 | GLOBAL | C_Vehicle | ClearParticleRef | ClearParticleRef__9C_VehicleFi |
| 0x00238d60 | 1292 | GLOBAL | C_Vehicle | LockParticle | LockParticle__9C_VehicleFii |
| 0x00239270 | 132 | GLOBAL | C_Vehicle | UnlockParticle | UnlockParticle__9C_VehicleFi |
| 0x00239300 | 436 | GLOBAL | C_Vehicle | AddParticle | AddParticle__9C_VehicleFRC14S_VEH_PARTICLE |
| 0x002394c0 | 96 | 13 | S_VEH_PARTICLE | {ctor} | __ct__14S_VEH_PARTICLEFRC14S_VEH_PARTICLE |
| 0x00239520 | 348 | 13 | gif::vector<14S_VEH_PARTICLE,8> | reserve | reserve__Q23gif26vector<14S_VEH_PARTICLE,8>Fi |
| 0x00239680 | 368 | GLOBAL | C_Vehicle | CloseParticles | CloseParticles__9C_VehicleFv |
| 0x002397f0 | 320 | GLOBAL | C_Vehicle | TickParticles | TickParticles__9C_VehicleFi |
| 0x00239930 | 192 | GLOBAL | C_Vehicle | GetWheelCamPos | GetWheelCamPos__9C_VehicleFiRC8S_vector |
| 0x002399f0 | 22924 | GLOBAL | C_Vehicle | InitCar | InitCar__9C_VehicleFP9S_CARINITUi |
| 0x0023f380 | 56 | GLOBAL | C_Vehicle | GetInit | GetInit__9C_VehicleFP9S_CARINIT |
| 0x0023f3c0 | 288 | GLOBAL | C_Vehicle | SetGear | SetGear__9C_VehicleFi |
| 0x0023f4e0 | 356 | GLOBAL | C_Vehicle | SetPower | SetPower__9C_VehicleFfb |
| 0x0023f650 | 344 | GLOBAL | C_Vehicle | SetBrake | SetBrake__9C_VehicleFf |
| 0x0023f7b0 | 120 | GLOBAL | C_Vehicle | SetClutch | SetClutch__9C_VehicleFf |
| 0x0023f830 | 168 | GLOBAL | C_Vehicle | SetSteer | SetSteer__9C_VehicleFf |
| 0x0023f8e0 | 80 | GLOBAL | C_Vehicle | SetSteeringLinearity | SetSteeringLinearity__9C_VehicleFf |
| 0x0023f930 | 392 | GLOBAL | C_Vehicle | SetEngineOn | SetEngineOn__9C_VehicleFbb |
| 0x0023fac0 | 52 | GLOBAL | C_Vehicle | SetSpeedLimit | SetSpeedLimit__9C_VehicleFf |
| 0x0023fb00 | 36 | GLOBAL | C_Vehicle | SetFuel | SetFuel__9C_VehicleFf |
| 0x0023fb30 | 96 | GLOBAL | C_Vehicle | SetHandbrake | SetHandbrake__9C_VehicleFbb |
| 0x0023fb90 | 76 | GLOBAL | C_Vehicle | EnableSounds | EnableSounds__9C_VehicleFb |
| 0x0023fbe0 | 72 | GLOBAL | C_Vehicle | SetCutTime | SetCutTime__9C_VehicleFi |
| 0x0023fc30 | 620 | GLOBAL | C_Vehicle | ExternalImpuls | ExternalImpuls__9C_VehicleFRC8S_vectorRC8S_vectorUiRC8S_vector |
| 0x0023fea0 | 244 | GLOBAL | C_Vehicle | ExternalForce | ExternalForce__9C_VehicleFRC8S_vectorRC8S_vectorUiRC8S_vector |
| 0x0023ffa0 | 968 | GLOBAL | C_Vehicle | DoCollision | DoCollision__9C_VehicleFPCvfRC8S_vectorRC8S_vectorRC8S_vectorf |
| 0x00240370 | 1144 | GLOBAL | C_Vehicle | DoCollisionStatic | DoCollisionStatic__9C_VehicleFRC8S_vectorRC8S_vectorRC8S_vector |
| 0x002407f0 | 268 | GLOBAL | C_Vehicle | GetRVelMassJ | GetRVelMassJ__9C_VehicleFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorPfP8S_vector |
| 0x00240900 | 8 | GLOBAL | C_Vehicle | GetHeadPos | GetHeadPos__9C_VehicleFv |
| 0x00240910 | 276 | GLOBAL | C_Vehicle | _GetMaxMomentByRot | _GetMaxMomentByRot__9C_VehicleFfPfib |
| 0x00240a30 | 156 | GLOBAL | C_Vehicle | SetSeatMass | SetSeatMass__9C_VehicleFif |
| 0x00240ad0 | 1016 | 13 | C_Vehicle | ComputeMass | ComputeMass__9C_VehicleFii |
| 0x00240ed0 | 432 | GLOBAL | C_Vehicle | CodePos | CodePos__9C_VehicleFiRC8S_vector |
| 0x00241080 | 348 | GLOBAL | C_Vehicle | DecodePos | DecodePos__9C_VehicleFiUi |
| 0x002411e0 | 296 | GLOBAL | C_Vehicle | CodeIndexDan | CodeIndexDan__9C_VehicleFi |
| 0x00241310 | 132 | GLOBAL | C_Vehicle | DecodeIndexDan | DecodeIndexDan__9C_VehicleFUi |
| 0x002413a0 | 44 | GLOBAL | C_Vehicle | GetRotations | GetRotations__9C_VehicleFv |
| 0x002413d0 | 20 | GLOBAL | C_Vehicle | IsVehicleUnstable | IsVehicleUnstable__9C_VehicleFv |
| 0x002413f0 | 20 | GLOBAL | C_Vehicle | GetVehicleSize | GetVehicleSize__9C_VehicleFb |
| 0x00241410 | 20 | GLOBAL | C_Vehicle | GetVehiclePivotPos | GetVehiclePivotPos__9C_VehicleFb |
| 0x00241430 | 96 | GLOBAL | C_Vehicle | LockVehicle | LockVehicle__9C_VehicleFb |
| 0x00241490 | 1180 | GLOBAL | C_Vehicle | SetProjectorSize | SetProjectorSize__9C_VehicleFUiRC8S_vector |
| 0x00241930 | 288 | GLOBAL | C_Vehicle | OpenDoor | OpenDoor__9C_VehicleFifb |
| 0x00241a50 | 116 | GLOBAL | C_Vehicle | IsDoor | IsDoor__9C_VehicleFi |
| 0x00241ad0 | 44 | GLOBAL | C_Vehicle | SetMaxDoorAngle | SetMaxDoorAngle__9C_VehicleFif |
| 0x00241b00 | 56 | GLOBAL | C_Vehicle | SetSpeedDoorOpen | SetSpeedDoorOpen__9C_VehicleFif |
| 0x00241b40 | 664 | GLOBAL | C_Vehicle | SetSpecialDynamicColls | SetSpecialDynamicColls__9C_VehicleFb |
| 0x00241de0 | 752 | GLOBAL | C_Vehicle | SetDynamicCollsOn | SetDynamicCollsOn__9C_VehicleFbb |
| 0x002420d0 | 836 | GLOBAL | C_Vehicle | UpdateImportantVariables | UpdateImportantVariables__9C_VehicleFv |
| 0x00242420 | 156 | GLOBAL | C_Vehicle | ClearBodyCollAddTrack | ClearBodyCollAddTrack__9C_VehicleFP14S_COLBODY_HEAD |
| 0x002424c0 | 300 | GLOBAL | C_Vehicle | AddProjectorLightsToSectors | AddProjectorLightsToSectors__9C_VehicleFv |
| 0x002425f0 | 244 | GLOBAL | C_Vehicle | DeleteProjectorLightsFromSectors | DeleteProjectorLightsFromSectors__9C_VehicleFv |
| 0x002426f0 | 148 | GLOBAL | C_Vehicle | AddSectorForProjectorLights | AddSectorForProjectorLights__9C_VehicleFP10I3D_sector |
| 0x00242790 | 148 | GLOBAL | C_Vehicle | AddSectorForSounds | AddSectorForSounds__9C_VehicleFP10I3D_sector |
| 0x00242830 | 1016 | GLOBAL | C_Vehicle | CalmVehicle | CalmVehicle__9C_VehicleFUifi |
| 0x00242c30 | 80 | GLOBAL | C_Vehicle | GetForwardForceMax | GetForwardForceMax__9C_VehicleFv |
| 0x00242c80 | 80 | GLOBAL | C_Vehicle | GetSideForceMax | GetSideForceMax__9C_VehicleFv |
| 0x00242cd0 | 2156 | GLOBAL | C_Vehicle | DuplicateDamageAndDeform | DuplicateDamageAndDeform__9C_VehicleFP9C_Vehicle |
| 0x00243540 | 136 | GLOBAL | C_Vehicle | GetVehicleStateSize | GetVehicleStateSize__9C_VehicleFv |
| 0x002435d0 | 2016 | GLOBAL | C_Vehicle | SaveVehicleState | SaveVehicleState__9C_VehicleFPv |
| 0x00243db0 | 5344 | GLOBAL | C_Vehicle | LoadVehicleState | LoadVehicleState__9C_VehicleFPv |
| 0x00245290 | 956 | GLOBAL | C_Vehicle | GetVehicleDamageDeformSize | GetVehicleDamageDeformSize__9C_VehicleFv |
| 0x00245650 | 2324 | GLOBAL | C_Vehicle | SaveVehicleDamageDeform | SaveVehicleDamageDeform__9C_VehicleFPv |
| 0x00245f70 | 3096 | GLOBAL | C_Vehicle | LoadVehicleDamageDeform | LoadVehicleDamageDeform__9C_VehicleFPv |
| 0x00246b90 | 284 | 13 | gif::vector<18S_DEFORM_ZONE_ORIG,8> | reserve | reserve__Q23gif30vector<18S_DEFORM_ZONE_ORIG,8>Fi |
| 0x00246cb0 | 264 | LOCAL |  | CbEnumObj__FP9I3D_frameUi | CbEnumObj__FP9I3D_frameUi |
| 0x00246dc0 | 20 | 13 | std::__vector_deleter<7S_SLAIF,Q23std19allocator<7S_SLAIF>> | {ctor} | __ct__Q23std54__vector_deleter<7S_SLAIF,Q23std19allocator<7S_SLAIF>>Fv |
| 0x00246de0 | 12 | LOCAL |  | CbEnumObj1__FP9I3D_frameUi | CbEnumObj1__FP9I3D_frameUi |
| 0x00246df0 | 80 | LOCAL |  | CbEnumObj6__FP9I3D_frameUi | CbEnumObj6__FP9I3D_frameUi |
| 0x00246e40 | 204 | LOCAL |  | IsParent__FP14S_COLBODY_HEADP14S_COLBODY_HEAD | IsParent__FP14S_COLBODY_HEADP14S_COLBODY_HEAD |
| 0x00246f10 | 672 | LOCAL |  | CbEnumBodyColl__FP9I3D_frameUi | CbEnumBodyColl__FP9I3D_frameUi |
| 0x002471b0 | 140 | LOCAL |  | EnumColSphere__FP9S_COL_POM | EnumColSphere__FP9S_COL_POM |
| 0x00247240 | 404 | LOCAL |  | CbEnumCollsActiveSphere__FP9I3D_frameUi | CbEnumCollsActiveSphere__FP9I3D_frameUi |
| 0x002473e0 | 652 | GLOBAL | C_Vehicle | PrepareBodyColls | PrepareBodyColls__9C_VehicleFP14S_COLBODY_HEAD |
| 0x00247670 | 1728 | LOCAL |  | CbEnumDeform__FP9I3D_frameUi | CbEnumDeform__FP9I3D_frameUi |
| 0x00247d30 | 668 | LOCAL |  | PrepareDeform__FP9I3D_framePQ23std58vector<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>>R13S_DEFORM_ZONEPCciUsRb | PrepareDeform__FP9I3D_framePQ23std58vector<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>>R13S_DEFORM_ZONEPCciUsRb |
| 0x00247fd0 | 8 | GLOBAL | C_Vehicle | SetCallbackACI | SetCallbackACI__9C_VehicleFPFPCvRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorPfP8S_vectorPfRb_b |
| 0x00247fe0 | 8 | GLOBAL | C_Vehicle | SetCallbackCC | SetCallbackCC__9C_VehicleFPFPCvPCvRC8S_vectorRC8S_vectorRC8S_vectorf_v |
| 0x00247ff0 | 8 | GLOBAL | C_Vehicle | SetCallbackCC1 | SetCallbackCC1__9C_VehicleFPFPCvPCvRC8S_vectorRC8S_vectorRC8S_vector_v |
| 0x00248000 | 8 | GLOBAL | C_Vehicle | SetCallbackAF | SetCallbackAF__9C_VehicleFPFPCvPCvRb_b |
| 0x00248010 | 8 | GLOBAL | C_Vehicle | SetCallbackWC | SetCallbackWC__9C_VehicleFPFPCvRC8S_vectorRC8S_vectorRC8S_vectorfRC8S_vectorPCvPCvb_v |
| 0x00248020 | 8 | GLOBAL | C_Vehicle | SetCallbackPD | SetCallbackPD__9C_VehicleFPFPCviRC8S_vectorPC8S_vector_b |
| 0x00248030 | 8 | GLOBAL | C_Vehicle | SetCallbackPDW | SetCallbackPDW__9C_VehicleFPFPCviRC8S_vectorPC8S_vector_b |
| 0x00248040 | 8 | GLOBAL | C_Vehicle | SetCallbackBG | SetCallbackBG__9C_VehicleFPFPCviRC8S_vectorPCc_v |
| 0x00248050 | 8 | GLOBAL | C_Vehicle | SetCallbackWU | SetCallbackWU__9C_VehicleFPFPCv_v |
| 0x00248060 | 8 | GLOBAL | C_Vehicle | SetCallbackCFB | SetCallbackCFB__9C_VehicleFPFPCvPCvRC8S_vectorRC8S_vector_PCv |
| 0x00248070 | 8 | GLOBAL | C_Vehicle | SetCallbackCFW | SetCallbackCFW__9C_VehicleFPFPCvPCvi_PCv |
| 0x00248080 | 8 | GLOBAL | C_Vehicle | SetCallbackDVP | SetCallbackDVP__9C_VehicleFPFPCvP9I3D_frameUiRC8S_vectorRC8S_vectorb_v |
| 0x00248090 | 8 | GLOBAL | C_Vehicle | SetCallbackVR | SetCallbackVR__9C_VehicleFPFPCv_v |
| 0x002480a0 | 8 | GLOBAL | C_Vehicle | SetCallbackLP | SetCallbackLP__9C_VehicleFPFR14S_VEH_PARTICLEi_b |
| 0x002480b0 | 8 | GLOBAL | C_Vehicle | SetCallbackCP | SetCallbackCP__9C_VehicleFPFii_i |
| 0x002480c0 | 8 | 13 | I3D_vertex | {ctor} | __ct__10I3D_vertexFv |
| 0x002480d0 | 1220 | WEAK | std::__vector_imp<7S_SLAIF,Q23std19allocator<7S_SLAIF>,0> | insert | insert__Q23std52__vector_imp<7S_SLAIF,Q23std19allocator<7S_SLAIF>,0>FP7S_SLAIFUiRC7S_SLAIF |
| 0x002485a0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std19allocator<7S_SLAIF>,Ui,1> | swap | swap__Q310Metrowerks7details53compressed_pair_imp<Q23std19allocator<7S_SLAIF>,Ui,1>FRQ310Metrowerks7details53compressed_pair_imp<Q23std19allocator<7S_SLAIF>,Ui,1> |
| 0x002485c0 | 1300 | WEAK | std::__vector_imp<7S_STRIP,Q23std19allocator<7S_STRIP>,0> | insert | insert__Q23std52__vector_imp<7S_STRIP,Q23std19allocator<7S_STRIP>,0>FP7S_STRIPUiRC7S_STRIP |
| 0x00248ae0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std19allocator<7S_STRIP>,Ui,1> | swap | swap__Q310Metrowerks7details53compressed_pair_imp<Q23std19allocator<7S_STRIP>,Ui,1>FRQ310Metrowerks7details53compressed_pair_imp<Q23std19allocator<7S_STRIP>,Ui,1> |
| 0x00248b00 | 1396 | WEAK | std::__vector_imp<12S_LPROJECTOR,Q23std25allocator<12S_LPROJECTOR>,0> | insert | insert__Q23std64__vector_imp<12S_LPROJECTOR,Q23std25allocator<12S_LPROJECTOR>,0>FP12S_LPROJECTORUiRC12S_LPROJECTOR |
| 0x00249080 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std25allocator<12S_LPROJECTOR>,Ui,1> | swap | swap__Q310Metrowerks7details59compressed_pair_imp<Q23std25allocator<12S_LPROJECTOR>,Ui,1>FRQ310Metrowerks7details59compressed_pair_imp<Q23std25allocator<12S_LPROJECTOR>,Ui,1> |
| 0x002490a0 | 72 | 13 | S_LPROJECTOR | operator= | __as__12S_LPROJECTORFRC12S_LPROJECTOR |
| 0x002490f0 | 1252 | WEAK | std::__vector_imp<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>,0> | insert | insert__Q23std66__vector_imp<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>,0>FP13S_DEFORM_ZONEUiRC13S_DEFORM_ZONE |
| 0x002495e0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_DEFORM_ZONE>,Ui,1> | swap | swap__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_DEFORM_ZONE>,Ui,1>FRQ310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_DEFORM_ZONE>,Ui,1> |
| 0x00249600 | 144 | 13 | S_DEFORM_ZONE | operator= | __as__13S_DEFORM_ZONEFRC13S_DEFORM_ZONE |
| 0x00249690 | 1364 | WEAK | std::__vector_imp<18S_DYN_COLL_MOVABLE,Q23std31allocator<18S_DYN_COLL_MOVABLE>,0> | insert | insert__Q23std76__vector_imp<18S_DYN_COLL_MOVABLE,Q23std31allocator<18S_DYN_COLL_MOVABLE>,0>FP18S_DYN_COLL_MOVABLEUiRC18S_DYN_COLL_MOVABLE |
| 0x00249bf0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18S_DYN_COLL_MOVABLE>,Ui,1> | swap | swap__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_DYN_COLL_MOVABLE>,Ui,1>FRQ310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18S_DYN_COLL_MOVABLE>,Ui,1> |
| 0x00249c10 | 1068 | WEAK | std::__vector_imp<19S_DYN_COLL_SETTINGS,Q23std32allocator<19S_DYN_COLL_SETTINGS>,0> | insert | insert__Q23std78__vector_imp<19S_DYN_COLL_SETTINGS,Q23std32allocator<19S_DYN_COLL_SETTINGS>,0>FP19S_DYN_COLL_SETTINGSUiRC19S_DYN_COLL_SETTINGS |
| 0x0024a040 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std32allocator<19S_DYN_COLL_SETTINGS>,Ui,1> | swap | swap__Q310Metrowerks7details66compressed_pair_imp<Q23std32allocator<19S_DYN_COLL_SETTINGS>,Ui,1>FRQ310Metrowerks7details66compressed_pair_imp<Q23std32allocator<19S_DYN_COLL_SETTINGS>,Ui,1> |
| 0x0024a060 | 80 | WEAK | std::__vector_deleter<7S_SLAIF,Q23std19allocator<7S_SLAIF>> | clear | clear__Q23std54__vector_deleter<7S_SLAIF,Q23std19allocator<7S_SLAIF>>Fv |
| 0x0024a0b0 | 104 | WEAK | std::__vector_deleter<7S_SLAIF,Q23std19allocator<7S_SLAIF>> | {dtor} | __dt__Q23std54__vector_deleter<7S_SLAIF,Q23std19allocator<7S_SLAIF>>Fv |
| 0x0024a120 | 80 | WEAK | std::__vector_deleter<7S_STRIP,Q23std19allocator<7S_STRIP>> | clear | clear__Q23std54__vector_deleter<7S_STRIP,Q23std19allocator<7S_STRIP>>Fv |
| 0x0024a170 | 104 | WEAK | std::__vector_deleter<7S_STRIP,Q23std19allocator<7S_STRIP>> | {dtor} | __dt__Q23std54__vector_deleter<7S_STRIP,Q23std19allocator<7S_STRIP>>Fv |
| 0x0024a1e0 | 80 | WEAK | std::__vector_deleter<12S_LPROJECTOR,Q23std25allocator<12S_LPROJECTOR>> | clear | clear__Q23std66__vector_deleter<12S_LPROJECTOR,Q23std25allocator<12S_LPROJECTOR>>Fv |
| 0x0024a230 | 104 | WEAK | std::__vector_deleter<12S_LPROJECTOR,Q23std25allocator<12S_LPROJECTOR>> | {dtor} | __dt__Q23std66__vector_deleter<12S_LPROJECTOR,Q23std25allocator<12S_LPROJECTOR>>Fv |
| 0x0024a2a0 | 80 | WEAK | std::__vector_deleter<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>> | clear | clear__Q23std68__vector_deleter<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>>Fv |
| 0x0024a2f0 | 104 | WEAK | std::__vector_deleter<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>> | {dtor} | __dt__Q23std68__vector_deleter<13S_DEFORM_ZONE,Q23std26allocator<13S_DEFORM_ZONE>>Fv |
| 0x0024a360 | 72 | WEAK | std::__vector_deleter<16S_COLBODY_EMITOR,Q23std29allocator<16S_COLBODY_EMITOR>> | clear | clear__Q23std74__vector_deleter<16S_COLBODY_EMITOR,Q23std29allocator<16S_COLBODY_EMITOR>>Fv |
| 0x0024a3b0 | 104 | WEAK | std::__vector_deleter<16S_COLBODY_EMITOR,Q23std29allocator<16S_COLBODY_EMITOR>> | {dtor} | __dt__Q23std74__vector_deleter<16S_COLBODY_EMITOR,Q23std29allocator<16S_COLBODY_EMITOR>>Fv |
| 0x0024a420 | 80 | WEAK | std::__vector_deleter<18S_DYN_COLL_MOVABLE,Q23std31allocator<18S_DYN_COLL_MOVABLE>> | clear | clear__Q23std78__vector_deleter<18S_DYN_COLL_MOVABLE,Q23std31allocator<18S_DYN_COLL_MOVABLE>>Fv |
| 0x0024a470 | 104 | WEAK | std::__vector_deleter<18S_DYN_COLL_MOVABLE,Q23std31allocator<18S_DYN_COLL_MOVABLE>> | {dtor} | __dt__Q23std78__vector_deleter<18S_DYN_COLL_MOVABLE,Q23std31allocator<18S_DYN_COLL_MOVABLE>>Fv |
| 0x0024a4e0 | 72 | WEAK | std::__vector_deleter<19S_DYN_COLL_SETTINGS,Q23std32allocator<19S_DYN_COLL_SETTINGS>> | clear | clear__Q23std80__vector_deleter<19S_DYN_COLL_SETTINGS,Q23std32allocator<19S_DYN_COLL_SETTINGS>>Fv |
| 0x0024a530 | 104 | WEAK | std::__vector_deleter<19S_DYN_COLL_SETTINGS,Q23std32allocator<19S_DYN_COLL_SETTINGS>> | {dtor} | __dt__Q23std80__vector_deleter<19S_DYN_COLL_SETTINGS,Q23std32allocator<19S_DYN_COLL_SETTINGS>>Fv |
| 0x0024a5a0 | 104 | 13 | gif::vector<18S_DEFORM_ZONE_ORIG,8> | {dtor} | __dt__Q23gif30vector<18S_DEFORM_ZONE_ORIG,8>Fv |
| 0x0024a610 | 140 | 13 | std::vector<P10I3D_sector,Q23std24allocator<P10I3D_sector>> | {dtor} | __dt__Q23std54vector<P10I3D_sector,Q23std24allocator<P10I3D_sector>>Fv |
| 0x0024a6a0 | 104 | 13 | gif::vector<14S_VEH_PARTICLE,8> | {dtor} | __dt__Q23gif26vector<14S_VEH_PARTICLE,8>Fv |
| 0x0024a710 | 6372 | GLOBAL | C_Vehicle | Slaif | Slaif__9C_VehicleFi |
| 0x0024c000 | 116 | GLOBAL | C_Vehicle | SlaifEnd | SlaifEnd__9C_VehicleFi |
| 0x0024c080 | 792 | GLOBAL | C_Vehicle | PrepareDeformZone | PrepareDeformZone__9C_VehicleFv |
| 0x0024c3a0 | 252 | GLOBAL | C_Vehicle | InitDeformGlassTexture | InitDeformGlassTexture__9C_VehicleFPCcPCc |
| 0x0024c4a0 | 1804 | GLOBAL | C_Vehicle | InitDeform | InitDeform__9C_VehicleFb |
| 0x0024cbb0 | 8 | 13 | I3D_material | UpdateTexture | UpdateTexture__12I3D_materialFv |
| 0x0024cbc0 | 368 | GLOBAL | C_Vehicle | InitDamage | InitDamage__9C_VehicleFv |
| 0x0024cd30 | 3108 | GLOBAL | C_Vehicle | UpdateDeformPart | UpdateDeformPart__9C_VehicleFiPC8S_vectorb |
| 0x0024d960 | 7876 | GLOBAL | C_Vehicle | Deform | Deform__9C_VehicleFRC8S_vectorRC8S_vectorffUiPC8S_vector |
| 0x0024f830 | 316 | GLOBAL | C_Vehicle | SwitchSpecialProjectorTexture | SwitchSpecialProjectorTexture__9C_VehicleFP9I3D_frameUs |
| 0x0024f970 | 472 | GLOBAL | C_Vehicle | DeformGlass | DeformGlass__9C_VehicleFiRC8S_vector |
| 0x0024fb50 | 340 | GLOBAL | C_Vehicle | DamageVehicle | DamageVehicle__9C_VehicleFf |
| 0x0024fcb0 | 2500 | GLOBAL | C_Vehicle | UpdateAxis | UpdateAxis__9C_VehicleFv |
| 0x00250680 | 968 | GLOBAL | C_Vehicle | DoDoors | DoDoors__9C_VehicleFf |
| 0x00250a50 | 92 | GLOBAL | C_Vehicle | OpenWindow | OpenWindow__9C_VehicleFib |
| 0x00250ab0 | 824 | GLOBAL | C_Vehicle | DoWindows | DoWindows__9C_VehicleFf |
| 0x00250df0 | 152 | GLOBAL | C_Vehicle | IsWindowOpen | IsWindowOpen__9C_VehicleFi |
| 0x00250e90 | 21472 | GLOBAL | C_Vehicle | DoLights | DoLights__9C_VehicleFf |
| 0x00256270 | 2464 | GLOBAL | C_Vehicle | UpdateStrips | UpdateStrips__9C_VehicleFf |
| 0x00256c10 | 1164 | GLOBAL | C_Vehicle | UpdateMotorParticles | UpdateMotorParticles__9C_VehicleFv |
| 0x002570a0 | 1736 | GLOBAL | C_Vehicle | UpdateSteeringWheels | UpdateSteeringWheels__9C_VehicleFf |
| 0x00257770 | 2472 | GLOBAL | C_Vehicle | UpdateRotationWheels | UpdateRotationWheels__9C_VehicleFf |
| 0x00258120 | 2100 | GLOBAL | C_Vehicle | UpdateWheelParticles | UpdateWheelParticles__9C_VehicleFv |
| 0x00258960 | 1592 | GLOBAL | C_Vehicle | UpdateExhaustParticles | UpdateExhaustParticles__9C_VehicleFv |
| 0x00258fa0 | 1312 | GLOBAL | C_Vehicle | UpdateBodyParticles | UpdateBodyParticles__9C_VehicleFv |
| 0x002594c0 | 8188 | GLOBAL | C_Vehicle | Tick | Tick__9C_VehicleFUi |
| 0x0025b4c0 | 8120 | GLOBAL | C_Vehicle | Update | Update__9C_VehicleFUi |
| 0x0025d480 | 168 | 13 | S_vector | Magnitude | Magnitude__8S_vectorCFv |
| 0x0025d530 | 904 | GLOBAL | C_Vehicle | MotorForce | MotorForce__9C_VehicleFff |
| 0x0025d8c0 | 416 | GLOBAL | C_Vehicle | ComputeMoment | ComputeMoment__9C_VehicleFff |
| 0x0025da60 | 1416 | GLOBAL | C_Vehicle | MotorRot | MotorRot__9C_VehicleFfff |
| 0x0025dff0 | 920 | GLOBAL | C_Vehicle | Engine | Engine__9C_VehicleFfff |
| 0x0025e390 | 17596 | GLOBAL | C_Vehicle | Move | Move__9C_VehicleFffffff |
| 0x00262850 | 9608 | GLOBAL | C_Vehicle | DoWheelsCollision | DoWheelsCollision__9C_VehicleFfRC8S_vectorb |
| 0x00264de0 | 8344 | GLOBAL | C_Vehicle | BodyCollision | BodyCollision__9C_VehicleFP14S_COLBODY_HEADR8S_vectorR8S_vectorRfR8S_vectorR8S_vectorUi |
| 0x00266e80 | 92 | 13 | std::vector<6S_SORT,Q23std18allocator<6S_SORT>> | {dtor} | __dt__Q23std42vector<6S_SORT,Q23std18allocator<6S_SORT>>Fv |
| 0x00266ee0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6S_SORT>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_SORT>,Ui,1>FUi |
| 0x00266ef0 | 48 | 13 |  | Min<f>__FRCfRCf | Min<f>__FRCfRCf |
| 0x00266f20 | 1052 | WEAK | std::__vector_imp<16S_COLBODY_EMITOR,Q23std29allocator<16S_COLBODY_EMITOR>,0> | insert | insert__Q23std72__vector_imp<16S_COLBODY_EMITOR,Q23std29allocator<16S_COLBODY_EMITOR>,0>FP16S_COLBODY_EMITORUiRC16S_COLBODY_EMITOR |
| 0x00267340 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std29allocator<16S_COLBODY_EMITOR>,Ui,1> | swap | swap__Q310Metrowerks7details63compressed_pair_imp<Q23std29allocator<16S_COLBODY_EMITOR>,Ui,1>FRQ310Metrowerks7details63compressed_pair_imp<Q23std29allocator<16S_COLBODY_EMITOR>,Ui,1> |
| 0x00267360 | 1068 | WEAK | std::__vector_imp<6S_SORT,Q23std18allocator<6S_SORT>,0> | insert | insert__Q23std50__vector_imp<6S_SORT,Q23std18allocator<6S_SORT>,0>FP6S_SORTUiRC6S_SORT |
| 0x00267790 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std18allocator<6S_SORT>,Ui,1> | swap | swap__Q310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_SORT>,Ui,1>FRQ310Metrowerks7details52compressed_pair_imp<Q23std18allocator<6S_SORT>,Ui,1> |
| 0x002677b0 | 72 | WEAK | std::__vector_deleter<6S_SORT,Q23std18allocator<6S_SORT>> | clear | clear__Q23std52__vector_deleter<6S_SORT,Q23std18allocator<6S_SORT>>Fv |
| 0x00267800 | 104 | WEAK | std::__vector_deleter<6S_SORT,Q23std18allocator<6S_SORT>> | {dtor} | __dt__Q23std52__vector_deleter<6S_SORT,Q23std18allocator<6S_SORT>>Fv |
| 0x00267870 | 1348 | GLOBAL | C_Vehicle | InitSounds | InitSounds__9C_VehicleFP9S_CARINIT |
| 0x00267dc0 | 2392 | GLOBAL | C_Vehicle | AddSoundsToSectors | AddSoundsToSectors__9C_VehicleFv |
| 0x00268720 | 2392 | GLOBAL | C_Vehicle | DeleteSoundsFromSectors | DeleteSoundsFromSectors__9C_VehicleFv |
| 0x00269080 | 1264 | GLOBAL | C_Vehicle | EngineSnd | EngineSnd__9C_VehicleFb |
| 0x00269570 | 5808 | GLOBAL | C_Vehicle | EngineFreq | EngineFreq__9C_VehicleFf |
| 0x0026ac20 | 1112 | GLOBAL | C_Vehicle | EnvironmentalSnd | EnvironmentalSnd__9C_VehicleFv |
| 0x0026b080 | 1120 | GLOBAL | C_Vehicle | CrashTyreSnd | CrashTyreSnd__9C_VehicleFb |
| 0x0026b4e0 | 1056 | GLOBAL | C_Vehicle | SkidSnd | SkidSnd__9C_VehicleFf |
| 0x0026b900 | 412 | GLOBAL | C_Vehicle | GearSnd | GearSnd__9C_VehicleFv |
| 0x0026baa0 | 248 | GLOBAL | C_Vehicle | HandBrake | HandBrake__9C_VehicleFv |
| 0x0026bba0 | 244 | GLOBAL | C_Vehicle | EngineSndNFS | EngineSndNFS__9C_VehicleFv |
| 0x0026bca0 | 328 | GLOBAL | C_Vehicle | HornSnd | HornSnd__9C_VehicleFb |
| 0x0026bdf0 | 328 | GLOBAL | C_Vehicle | SirenSnd | SirenSnd__9C_VehicleFb |
| 0x0026bf40 | 252 | GLOBAL | C_Vehicle | TyreBreakSnd | TyreBreakSnd__9C_VehicleFi |
| 0x0026c040 | 4256 | GLOBAL | C_Vehicle | CrashSnd | CrashSnd__9C_VehicleFfb |
| 0x0026d0e0 | 376 | GLOBAL | C_Vehicle | WheelSnd | WheelSnd__9C_VehicleFf |
| 0x0026d260 | 656 | GLOBAL | C_Vehicle | DoorSnd | DoorSnd__9C_VehicleFib |
| 0x0026d4f0 | 548 | GLOBAL | C_Vehicle | MotorSoundOff | MotorSoundOff__9C_VehicleFb |
| 0x0026d720 | 1764 | GLOBAL | C_Vehicle | SoundOff | SoundOff__9C_VehicleFv |
| 0x0026de10 | 1156 | GLOBAL | C_Vehicle | OpenSnd | OpenSnd__9C_VehicleFP9I3D_soundPCcUi |
| 0x0026e2a0 | 1368 | GLOBAL | C_Vehicle | SoundInit | SoundInit__9C_VehicleFPcPP9I3D_soundUiffffbb |
| 0x0026e800 | 2760 | GLOBAL | C_Vehicle | ReleaseSounds | ReleaseSounds__9C_VehicleFv |
| 0x0026f2d0 | 240 | GLOBAL | C_wagon | {ctor} | __ct__7C_wagonFv |
| 0x0026f3c0 | 180 | GLOBAL | C_wagon | {dtor} | __dt__7C_wagonFv |
| 0x0026f480 | 32 | GLOBAL | C_wagon | Init | Init__7C_wagonFP9I3D_frame |
| 0x0026f4a0 | 8 | GLOBAL | C_wagon | SaveData | SaveData__7C_wagonFP7C_chunk |
| 0x0026f4b0 | 8 | GLOBAL | C_wagon | LoadData | LoadData__7C_wagonFP7C_chunk |
| 0x0026f4c0 | 64 | GLOBAL | C_wagon | SaveGameGetSize | SaveGameGetSize__7C_wagonFv |
| 0x0026f500 | 472 | GLOBAL | C_wagon | SaveGameSave | SaveGameSave__7C_wagonFPPv |
| 0x0026f6e0 | 944 | GLOBAL | C_wagon | SaveGameLoad | SaveGameLoad__7C_wagonFPPv |
| 0x0026fa90 | 8 | GLOBAL | C_wagon | DuplicateFrom | DuplicateFrom__7C_wagonFP7C_actor |
| 0x0026faa0 | 8 | GLOBAL | C_wagon | Hit | Hit__7C_wagonF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x0026fab0 | 700 | GLOBAL | C_wagon | CreateBackVector | CreateBackVector__7C_wagonFR8S_vector |
| 0x0026fd70 | 1308 | GLOBAL | C_wagon | CreateForwardVector | CreateForwardVector__7C_wagonFUif |
| 0x00270290 | 36 | GLOBAL | C_wagon | IsStopPoint | IsStopPoint__7C_wagonFUi |
| 0x002702c0 | 800 | GLOBAL | C_wagon | MustStop | MustStop__7C_wagonFv |
| 0x002705e0 | 1872 | GLOBAL | C_wagon | FindBackPos | FindBackPos__7C_wagonFR8S_vectorfPi |
| 0x00270d30 | 1040 | GLOBAL | C_wagon | SpringProcess | SpringProcess__7C_wagonFUiiR8S_vectorR8S_vectorR8S_vectorR8S_vector |
| 0x00271140 | 5316 | GLOBAL | C_wagon | AI | AI__7C_wagonFUi |
| 0x00272610 | 8 | 13 | std::__vector_imp<Ui,Q23std13allocator<Ui>,1> | size | size__Q23std40__vector_imp<Ui,Q23std13allocator<Ui>,1>CFv |
| 0x00272620 | 1328 | GLOBAL | C_wagon | Update | Update__7C_wagonFUi |
| 0x00272b50 | 8 | GLOBAL | C_wagon | NetDirect | NetDirect__7C_wagonFUi |
| 0x00272b60 | 5288 | GLOBAL | C_wagon | GameInit | GameInit__7C_wagonFv |
| 0x00274010 | 488 | 13 | C_wagon::S_hlp$3866C_wagon_cpp | cbEnumFrame | cbEnumFrame__Q27C_wagon21S_hlp$3866C_wagon_cppFP9I3D_frameRQ23std50vector<P9I3D_frame,Q23std22allocator<P9I3D_frame>> |
| 0x00274200 | 376 | GLOBAL | C_wagon | GameDone | GameDone__7C_wagonFv |
| 0x00274380 | 208 | GLOBAL | C_wagon | RecompileNewPath | RecompileNewPath__7C_wagonFv |
| 0x00274450 | 220 | GLOBAL | C_console | {ctor} | __ct__9C_consoleFv |
| 0x00274530 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14S_console_line>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_console_line>,Ui,1>FUi |
| 0x00274540 | 108 | GLOBAL | C_console | {dtor} | __dt__9C_consoleFv |
| 0x002745b0 | 168 | GLOBAL | C_console | Sprintf | Sprintf__9C_consoleFiPCce |
| 0x00274660 | 328 | GLOBAL | C_console | OutText | OutText__9C_consoleFPciiii |
| 0x002747b0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14S_console_line>,Ui,1> | first | first__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_console_line>,Ui,1>Fv |
| 0x002747c0 | 112 | GLOBAL | C_console | Clear | Clear__9C_consoleFv |
| 0x00274830 | 380 | GLOBAL | C_console | Tick | Tick__9C_consoleFUi |
| 0x002749b0 | 568 | GLOBAL | C_console | RenderIndicators | RenderIndicators__9C_consoleFv |
| 0x00274bf0 | 1512 | GLOBAL | C_console | Render | Render__9C_consoleFv |
| 0x002751e0 | 188 | GLOBAL |  | ConsoleReg__FP9C_consoleUi | ConsoleReg__FP9C_consoleUi |
| 0x002752a0 | 176 | GLOBAL |  | ConsoleUnReg__FP9C_console | ConsoleUnReg__FP9C_console |
| 0x00275350 | 372 | GLOBAL |  | ConsoleRender__Fv | ConsoleRender__Fv |
| 0x002754d0 | 148 | GLOBAL |  | CreateFontTexture__FPP8ITexture | CreateFontTexture__FPP8ITexture |
| 0x00275570 | 8 | GLOBAL |  | ConsolesInit__Fv | ConsolesInit__Fv |
| 0x00275580 | 40 | GLOBAL |  | ConsolesDone__Fv | ConsolesDone__Fv |
| 0x002755b0 | 80 | WEAK | std::__vector_deleter<14S_console_line,Q23std27allocator<14S_console_line>> | clear | clear__Q23std70__vector_deleter<14S_console_line,Q23std27allocator<14S_console_line>>Fv |
| 0x00275600 | 316 | WEAK | std::__vector_imp<14S_console_line,Q23std27allocator<14S_console_line>,0> | init | init__Q23std68__vector_imp<14S_console_line,Q23std27allocator<14S_console_line>,0>FUiRC14S_console_line |
| 0x00275740 | 208 | WEAK | std::__vector_imp<14S_console_line,Q23std27allocator<14S_console_line>,0> | erase | erase__Q23std68__vector_imp<14S_console_line,Q23std27allocator<14S_console_line>,0>FP14S_console_line |
| 0x00275810 | 64 | 13 | S_console_line | operator= | __as__14S_console_lineFRC14S_console_line |
| 0x00275850 | 1388 | WEAK | std::__vector_imp<14S_console_line,Q23std27allocator<14S_console_line>,0> | insert | insert__Q23std68__vector_imp<14S_console_line,Q23std27allocator<14S_console_line>,0>FP14S_console_lineUiRC14S_console_line |
| 0x00275dc0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14S_console_line>,Ui,1> | swap | swap__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_console_line>,Ui,1>FRQ310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_console_line>,Ui,1> |
| 0x00275de0 | 148 | WEAK | std::__vector_imp<14S_con_register,Q23std27allocator<14S_con_register>,0> | erase | erase__Q23std68__vector_imp<14S_con_register,Q23std27allocator<14S_con_register>,0>FP14S_con_register |
| 0x00275e80 | 1172 | WEAK | std::__vector_imp<14S_con_register,Q23std27allocator<14S_con_register>,0> | insert | insert__Q23std68__vector_imp<14S_con_register,Q23std27allocator<14S_con_register>,0>FP14S_con_registerUiRC14S_con_register |
| 0x00276320 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14S_con_register>,Ui,1> | swap | swap__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_con_register>,Ui,1>FRQ310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_con_register>,Ui,1> |
| 0x00276340 | 104 | WEAK | std::__vector_deleter<14S_console_line,Q23std27allocator<14S_console_line>> | {dtor} | __dt__Q23std70__vector_deleter<14S_console_line,Q23std27allocator<14S_console_line>>Fv |
| 0x002763b0 | 104 | WEAK | std::__vector_deleter<14S_con_register,Q23std27allocator<14S_con_register>> | {dtor} | __dt__Q23std70__vector_deleter<14S_con_register,Q23std27allocator<14S_con_register>>Fv |
| 0x00276420 | 104 | WEAK | std::__vector_deleter<10S_vertex2D,Q23std23allocator<10S_vertex2D>> | {dtor} | __dt__Q23std62__vector_deleter<10S_vertex2D,Q23std23allocator<10S_vertex2D>>Fv |
| 0x00276490 | 80 | WEAK | std::__vector_deleter<14S_con_register,Q23std27allocator<14S_con_register>> | clear | clear__Q23std70__vector_deleter<14S_con_register,Q23std27allocator<14S_con_register>>Fv |
| 0x002764e0 | 72 | WEAK | std::__vector_deleter<10S_vertex2D,Q23std23allocator<10S_vertex2D>> | clear | clear__Q23std62__vector_deleter<10S_vertex2D,Q23std23allocator<10S_vertex2D>>Fv |
| 0x00276530 | 92 | 13 | std::vector<10S_vertex2D,Q23std23allocator<10S_vertex2D>> | {dtor} | __dt__Q23std52vector<10S_vertex2D,Q23std23allocator<10S_vertex2D>>Fv |
| 0x00276590 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std23allocator<10S_vertex2D>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details57compressed_pair_imp<Q23std23allocator<10S_vertex2D>,Ui,1>FUi |
| 0x002765a0 | 92 | 13 | std::vector<14S_con_register,Q23std27allocator<14S_con_register>> | {dtor} | __dt__Q23std60vector<14S_con_register,Q23std27allocator<14S_con_register>>Fv |
| 0x00276600 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14S_con_register>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14S_con_register>,Ui,1>FUi |
| 0x00276610 | 712 | GLOBAL | g_collision | PreallocPrepare | PreallocPrepare__11g_collisionFPCcPCc |
| 0x002768e0 | 284 | GLOBAL | g_collision | DeleteCollsLevel | DeleteCollsLevel__11g_collisionFb |
| 0x00276a00 | 988 | GLOBAL |  | GrantCoherence__FP15tCollisionLevel | GrantCoherence__FP15tCollisionLevel |
| 0x00276de0 | 36 | 13 | S_vector | GetNormal | GetNormal__8S_vectorFRC8S_vectorRC8S_vectorRC8S_vector |
| 0x00276e10 | 4252 | GLOBAL | g_collision | LoadCollision | LoadCollision__11g_collisionFPCcb |
| 0x00277eb0 | 392 | GLOBAL | g_collision | DynOptimizeTick | DynOptimizeTick__11g_collisionFv |
| 0x00278040 | 260 | 13 | gif::vector<P15tDynamicPolicko,8> | reserve | reserve__Q23gif28vector<P15tDynamicPolicko,8>Fi |
| 0x00278150 | 484 | GLOBAL | g_collision | DynInsert | DynInsert__11g_collisionFP18tDynamicCollObject |
| 0x00278340 | 384 | GLOBAL | g_collision | DynErase | DynErase__11g_collisionFP18tDynamicCollObject |
| 0x002784c0 | 760 | GLOBAL | g_collision | DynUpdate | DynUpdate__11g_collisionFP18tDynamicCollObject |
| 0x002787c0 | 168 | GLOBAL | g_collision | DynSetNextCollOwner | DynSetNextCollOwner__11g_collisionFP18tDynamicCollObjectPFP18tDynamicCollObjectP13tCSphereParam_13gcCallbackRet |
| 0x00278870 | 160 | GLOBAL | g_collision | DynSetNextCollOwnerMult | DynSetNextCollOwnerMult__11g_collisionFPP18tDynamicCollObjectUiPFP18tDynamicCollObjectP13tCSphereParam_13gcCallbackRet |
| 0x00278910 | 1172 | GLOBAL | g_collision | DynWipeOut | DynWipeOut__11g_collisionFv |
| 0x00278db0 | 96 | GLOBAL | g_collision | DynGetIn | DynGetIn__11g_collisionFP18tDynamicCollObjectPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>> |
| 0x00278e10 | 128 | GLOBAL | g_collision | DynGetOut | DynGetOut__11g_collisionFP18tDynamicCollObjectPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>> |
| 0x00278e90 | 912 | GLOBAL | g_collision | BoxInDynamic | BoxInDynamic__11g_collisionFR10tCBoxParam |
| 0x00279220 | 748 | GLOBAL | g_collision | BoxInNonFace | BoxInNonFace__11g_collisionFP16g_volume_trifaceR10tCBoxParam |
| 0x00279510 | 1092 | GLOBAL | g_collision | BoxInTreeLevel | BoxInTreeLevel__11g_collisionFP14tCollTreeLevelR10tCBoxParam |
| 0x00279960 | 2648 | GLOBAL | g_collision | _TestDBox | _TestDBox__11g_collisionFRC8I3D_bboxRC8S_matrixRC8S_matrixRC8S_vectorRC8S_vectorRC8S_vectorUiUi |
| 0x0027a3c0 | 260 | WEAK | std::__vector_pod<Ui,Q23std13allocator<Ui>> | reserve | reserve__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>FUi |
| 0x0027a4d0 | 164 | 13 | g_collision | {dtor} | __dt__11g_collisionFv |
| 0x0027a580 | 2080 | GLOBAL | g_collision | LineInDynamics | LineInDynamics__11g_collisionFR11tCLineParam |
| 0x0027ada0 | 2168 | GLOBAL | g_collision | LineInDynamicsInnerDyn | LineInDynamicsInnerDyn__11g_collisionFR11tCLineParam |
| 0x0027b620 | 1524 | GLOBAL | g_collision | LineInAABox | LineInAABox__11g_collisionFRC8S_vectorRC8S_vectorR11tCLineParam |
| 0x0027bc20 | 788 | GLOBAL | g_collision | LineInAABoxInverted | LineInAABoxInverted__11g_collisionFRC8S_vectorRC8S_vectorR11tCLineParam |
| 0x0027bf40 | 1084 | GLOBAL | g_collision | LineInAACapsule | LineInAACapsule__11g_collisionFffR11tCLineParam |
| 0x0027c380 | 2404 | GLOBAL | g_collision | LineInNonface | LineInNonface__11g_collisionFP16g_volume_trifaceR11tCLineParam |
| 0x0027ccf0 | 1896 | GLOBAL | g_collision | LineInTreeLevel | LineInTreeLevel__11g_collisionFP14tCollTreeLevelR11tCLineParam |
| 0x0027d460 | 1204 | GLOBAL | g_collision | _TestLineV | _TestLineV__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi |
| 0x0027d920 | 1204 | GLOBAL | g_collision | _TestLineVInnerDyn | _TestLineVInnerDyn__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi |
| 0x0027dde0 | 4648 | GLOBAL | g_collision | TestLineHStatic | TestLineHStatic__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUi |
| 0x0027f010 | 744 | GLOBAL | g_collision | TestLineHDynamic | TestLineHDynamic__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi |
| 0x0027f300 | 4952 | GLOBAL | g_collision | _TestLineH | _TestLineH__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi |
| 0x00280660 | 1444 | GLOBAL | g_collision | LineInVehicle | LineInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>R11tCLineParam |
| 0x00280c10 | 636 | GLOBAL | g_collision | TestLineInVehicle | TestLineInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>RC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUi |
| 0x00280e90 | 2444 | GLOBAL | g_collision | LocalityInNonface | LocalityInNonface__11g_collisionFP16g_volume_trifaceRC8S_vectorRC8S_vectorRC8S_vectorfRQ23gif31vector<P18g_collision_header,8> |
| 0x00281820 | 260 | 13 | gif::vector<P18g_collision_header,8> | reserve | reserve__Q23gif31vector<P18g_collision_header,8>Fi |
| 0x00281930 | 236 | GLOBAL | g_collision | LocalityInTreeLevel | LocalityInTreeLevel__11g_collisionFP14tCollTreeLevelRC8S_vectorRC8S_vectorRC8S_vectorfUiUiRQ23gif31vector<P18g_collision_header,8> |
| 0x00281a20 | 652 | GLOBAL | g_collision | FindStaticCollsInLocality | FindStaticCollsInLocality__11g_collisionFRC8S_vectorfUiUiRQ23gif31vector<P18g_collision_header,8> |
| 0x00281cb0 | 1676 | GLOBAL | g_collision | FindCollsByFrame | FindCollsByFrame__11g_collisionFP9I3D_frameRQ23gif31vector<P18g_collision_header,8> |
| 0x00282340 | 740 | GLOBAL | g_collision | _TestLineS | _TestLineS__11g_collisionFRC8S_vectorRC8S_vectorUi |
| 0x00282630 | 1540 | GLOBAL | g_collision | LineInSTreeLevel | LineInSTreeLevel__11g_collisionFP14tCollTreeLevelR11tCLineParam |
| 0x00282c40 | 1684 | GLOBAL | g_collision | LineInSNonface | LineInSNonface__11g_collisionFP16g_volume_trifaceR11tCLineParam |
| 0x002832e0 | 3424 | GLOBAL | g_collision | FindDynamicCollsInLine | FindDynamicCollsInLine__11g_collisionFRC8S_vectorRC8S_vectorRQ23gif31vector<P18g_collision_header,8>Ui |
| 0x00284040 | 4188 | GLOBAL | g_collision | SphereInDynamics | SphereInDynamics__11g_collisionFR13tCSphereParam |
| 0x002850a0 | 1536 | GLOBAL | g_collision | SphereInEdge | SphereInEdge__11g_collisionFRC8S_vectorRC8S_vectorR13tCSphereParamRUiUiUi |
| 0x002856a0 | 3936 | GLOBAL | g_collision | SphereInFace | SphereInFace__11g_collisionFP16g_volume_trifaceR13tCSphereParamRi |
| 0x00286600 | 22004 | GLOBAL | g_collision | SphereInAABox | SphereInAABox__11g_collisionFRC8S_vectorRC8S_vectorR13tCSphereParam |
| 0x0028bc00 | 2348 | GLOBAL | g_collision | SphereInAACapsule | SphereInAACapsule__11g_collisionFffR13tCSphereParam |
| 0x0028c530 | 10080 | GLOBAL | g_collision | SphereInNonface | SphereInNonface__11g_collisionFP16g_volume_trifaceR13tCSphereParam |
| 0x0028ec90 | 760 | GLOBAL | g_collision | SphereInTreeLevel | SphereInTreeLevel__11g_collisionFP14tCollTreeLevelR13tCSphereParam |
| 0x0028ef90 | 2740 | GLOBAL | g_collision | _TestSphereB | _TestSphereB__11g_collisionFRC8S_vectorRC8S_vectorfP8S_vectorP8S_vectorUiUi |
| 0x0028fa50 | 1208 | GLOBAL | g_collision | TestCurveCurve | TestCurveCurve__11g_collisionFPCQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>fPCQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>f |
| 0x0028ff10 | 2752 | GLOBAL | g_collision | SphereInVehicle | SphereInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>R13tCSphereParam |
| 0x002909d0 | 896 | GLOBAL | g_collision | TestSphereInVehicle | TestSphereInVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>RC8S_vectorRC8S_vectorfP8S_vectorP8S_vectorUiUi |
| 0x00290d50 | 132 | GLOBAL | G_Camera | {ctor} | __ct__8G_CameraFv |
| 0x00290de0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15G_CameraPushPop>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15G_CameraPushPop>,Ui,1>FUi |
| 0x00290df0 | 132 | GLOBAL | G_Camera | {dtor} | __dt__8G_CameraFv |
| 0x00290e80 | 172 | GLOBAL | G_Camera | Reset | Reset__8G_CameraFv |
| 0x00290f30 | 64 | GLOBAL | G_Camera | GameDone | GameDone__8G_CameraFv |
| 0x00290f70 | 60 | GLOBAL | G_Camera | SetCamera | SetCamera__8G_CameraFP10I3D_camera |
| 0x00290fb0 | 60 | GLOBAL | G_Camera | SetPlayer | SetPlayer__8G_CameraFP7C_human |
| 0x00290ff0 | 104 | GLOBAL | G_Camera | SetCar | SetCar__8G_CameraFP5C_car |
| 0x00291060 | 68 | GLOBAL | G_Camera | Link | Link__8G_CameraFP9I3D_frame |
| 0x002910b0 | 276 | GLOBAL | G_Camera | SetSwing | SetSwing__8G_CameraFbfP9I3D_frame |
| 0x002911d0 | 200 | GLOBAL | G_Camera | LookAround | LookAround__8G_CameraFi |
| 0x002912a0 | 40 | GLOBAL | G_Camera | LookFromEyes | LookFromEyes__8G_CameraFv |
| 0x002912d0 | 256 | GLOBAL | G_Camera | SetMode | SetMode__8G_CameraFUib |
| 0x002913d0 | 188 | GLOBAL | G_Camera | LezeDoToyoty | LezeDoToyoty__8G_CameraFb |
| 0x00291490 | 556 | GLOBAL | G_Camera | CalcUpDown | CalcUpDown__8G_CameraFRC8S_vector |
| 0x002916c0 | 19556 | GLOBAL | G_Camera | Tick | Tick__8G_CameraFUi |
| 0x00296330 | 948 | GLOBAL | G_Camera | WantPosCar | WantPosCar__8G_CameraFRC8S_vectorRC8S_vectorf |
| 0x002966f0 | 2588 | GLOBAL | G_Camera | WantPos | WantPos__8G_CameraFRC8S_vectorRC8S_vectorRC8S_vectorf |
| 0x00297110 | 1860 | GLOBAL | G_Camera | Begin | Begin__8G_CameraFv |
| 0x00297860 | 300 | GLOBAL | G_Camera | End | End__8G_CameraFv |
| 0x00297990 | 92 | GLOBAL | G_Camera | EnableFreeLook | EnableFreeLook__8G_CameraFbb |
| 0x002979f0 | 552 | GLOBAL | G_Camera | FreeLook | FreeLook__8G_CameraFff |
| 0x00297c20 | 596 | GLOBAL | G_Camera | FreeLookBase | FreeLookBase__8G_CameraFffR8S_vectorR8S_vectorR8S_vector |
| 0x00297e80 | 212 | GLOBAL | G_Camera | LockAt | LockAt__8G_CameraFRC8S_vectorRC8S_vector |
| 0x00297f60 | 1568 | GLOBAL | G_Camera | CumNaNe | CumNaNe__8G_CameraFP7C_humanP7C_human |
| 0x00298580 | 44 | GLOBAL | G_Camera | Unlock | Unlock__8G_CameraFb |
| 0x002985b0 | 244 | GLOBAL | G_Camera | HardLockAt | HardLockAt__8G_CameraFRC8S_vectorRC8S_vector |
| 0x002986b0 | 8 | GLOBAL | G_Camera | HardUnlock | HardUnlock__8G_CameraFv |
| 0x002986c0 | 104 | GLOBAL | G_Camera | PushMode | PushMode__8G_CameraFv |
| 0x00298730 | 224 | GLOBAL | G_Camera | PopMode | PopMode__8G_CameraFb |
| 0x00298810 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15G_CameraPushPop>,Ui,1> | first | first__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15G_CameraPushPop>,Ui,1>Fv |
| 0x00298820 | 200 | GLOBAL | G_Camera | DialogLook | DialogLook__8G_CameraFP7C_humanP7C_human |
| 0x002988f0 | 632 | GLOBAL | G_Camera | DialogStart | DialogStart__8G_CameraFPP7C_humanUi |
| 0x00298b70 | 348 | 13 | gif::vector<14G_CameraDialog,8> | reserve | reserve__Q23gif26vector<14G_CameraDialog,8>Fi |
| 0x00298cd0 | 176 | GLOBAL | G_Camera | DialogStop | DialogStop__8G_CameraFv |
| 0x00298d80 | 16 | GLOBAL | G_Camera | IsInDialog | IsInDialog__8G_CameraFv |
| 0x00298d90 | 136 | GLOBAL | G_Camera | AddRaceCamera | AddRaceCamera__8G_CameraFP12G_RaceCamera |
| 0x00298e20 | 108 | GLOBAL | G_Camera | RemoveRaceCamera | RemoveRaceCamera__8G_CameraFP12G_RaceCamera |
| 0x00298e90 | 436 | GLOBAL | G_Camera | InitRaceCameras | InitRaceCameras__8G_CameraFv |
| 0x00299050 | 156 | GLOBAL | G_Camera | SetLookTo | SetLookTo__8G_CameraFRC8S_vectorP9I3D_frame |
| 0x002990f0 | 88 | GLOBAL | G_Camera | SetSniperFov | SetSniperFov__8G_CameraFf |
| 0x00299150 | 108 | GLOBAL | G_Camera | SetSniperMode | SetSniperMode__8G_CameraFb |
| 0x002991c0 | 72 | WEAK | std::__vector_deleter<15G_CameraPushPop,Q23std28allocator<15G_CameraPushPop>> | clear | clear__Q23std72__vector_deleter<15G_CameraPushPop,Q23std28allocator<15G_CameraPushPop>>Fv |
| 0x00299210 | 1180 | WEAK | std::__vector_imp<15G_CameraPushPop,Q23std28allocator<15G_CameraPushPop>,0> | insert | insert__Q23std70__vector_imp<15G_CameraPushPop,Q23std28allocator<15G_CameraPushPop>,0>FP15G_CameraPushPopUiRC15G_CameraPushPop |
| 0x002996b0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15G_CameraPushPop>,Ui,1> | swap | swap__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15G_CameraPushPop>,Ui,1>FRQ310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15G_CameraPushPop>,Ui,1> |
| 0x002996d0 | 368 | WEAK | std::__vector_imp<15G_CameraPushPop,Q23std28allocator<15G_CameraPushPop>,0> | reserve | reserve__Q23std70__vector_imp<15G_CameraPushPop,Q23std28allocator<15G_CameraPushPop>,0>FUi |
| 0x00299840 | 104 | WEAK | std::__vector_deleter<15G_CameraPushPop,Q23std28allocator<15G_CameraPushPop>> | {dtor} | __dt__Q23std72__vector_deleter<15G_CameraPushPop,Q23std28allocator<15G_CameraPushPop>>Fv |
| 0x002998b0 | 104 | 13 | gif::vector<14G_CameraDialog,8> | {dtor} | __dt__Q23gif26vector<14G_CameraDialog,8>Fv |
| 0x00299920 | 60 | GLOBAL | g_collision | GameDone | GameDone__11g_collisionFb |
| 0x00299960 | 12 | GLOBAL | g_collision | TestLineV | TestLineV__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi |
| 0x00299970 | 12 | GLOBAL | g_collision | TestLineVInnerDyn | TestLineVInnerDyn__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi |
| 0x00299980 | 8 | GLOBAL | g_collision | TestLineH | TestLineH__11g_collisionFRC8S_vectorRC8S_vectorP8S_vectorP8S_vectorUiUi |
| 0x00299990 | 12 | GLOBAL | g_collision | TestLineS | TestLineS__11g_collisionFRC8S_vectorRC8S_vectorUi |
| 0x002999a0 | 12 | GLOBAL | g_collision | TestSphereB | TestSphereB__11g_collisionFRC8S_vectorRC8S_vectorfP8S_vectorP8S_vectorUiUi |
| 0x002999b0 | 32 | GLOBAL | g_collision | TestDBox | TestDBox__11g_collisionFRC8I3D_bboxRC8S_matrixRC8S_matrixRC8S_vectorRC8S_vectorRC8S_vectorUiUi |
| 0x002999d0 | 2376 | GLOBAL | g_collision | TestPanak2 | TestPanak2__11g_collisionFP9tgcPanak2 |
| 0x0029a320 | 2324 | GLOBAL | g_collision | TestPanak2InVehicle | TestPanak2InVehicle__11g_collisionFPQ23std70vector<P18tDynamicCollObject,Q23std32allocator<P18tDynamicCollObject>>P9tgcPanak2 |
| 0x0029ac40 | 52 | GLOBAL |  | DabingInit__Fv | DabingInit__Fv |
| 0x0029ac80 | 164 | GLOBAL |  | DabingDone__Fv | DabingDone__Fv |
| 0x0029ad30 | 84 | GLOBAL |  | DabingGetFreeChanel__Fv | DabingGetFreeChanel__Fv |
| 0x0029ad90 | 1204 | GLOBAL |  | DabingSetHuman__FiP7C_human | DabingSetHuman__FiP7C_human |
| 0x0029b250 | 644 | GLOBAL |  | DabingSetModel__FiP9I3D_model | DabingSetModel__FiP9I3D_model |
| 0x0029b4e0 | 628 | GLOBAL |  | DabingInvalidate__FP7C_human | DabingInvalidate__FP7C_human |
| 0x0029b760 | 2148 | GLOBAL |  | DabingSpeak__FiUibUi | DabingSpeak__FiUibUi |
| 0x0029bfd0 | 108 | GLOBAL |  | DabingIsSpeaking__Fi | DabingIsSpeaking__Fi |
| 0x0029c040 | 140 | GLOBAL |  | DabingStop__Fi | DabingStop__Fi |
| 0x0029c0d0 | 3576 | GLOBAL |  | DabingTick__FUi | DabingTick__FUi |
| 0x0029ced0 | 284 | GLOBAL |  | DabingPauseSounds__Fbb | DabingPauseSounds__Fbb |
| 0x0029cff0 | 308 | GLOBAL |  | DabingPreLoad__FPCc | DabingPreLoad__FPCc |
| 0x0029d130 | 44 | GLOBAL |  | DabingPreUnload__Fv | DabingPreUnload__Fv |
| 0x0029d160 | 176 | WEAK | std::__vector_imp<16G_DabingMorphObj,Q23std29allocator<16G_DabingMorphObj>,0> | erase | erase__Q23std72__vector_imp<16G_DabingMorphObj,Q23std29allocator<16G_DabingMorphObj>,0>FP16G_DabingMorphObj |
| 0x0029d210 | 64 | 13 | G_DabingMorphObj | operator= | __as__16G_DabingMorphObjFRC16G_DabingMorphObj |
| 0x0029d250 | 72 | WEAK | std::__vector_deleter<16G_DabingMorphObj,Q23std29allocator<16G_DabingMorphObj>> | clear | clear__Q23std74__vector_deleter<16G_DabingMorphObj,Q23std29allocator<16G_DabingMorphObj>>Fv |
| 0x0029d2a0 | 104 | WEAK | std::__vector_deleter<16G_DabingMorphObj,Q23std29allocator<16G_DabingMorphObj>> | {dtor} | __dt__Q23std74__vector_deleter<16G_DabingMorphObj,Q23std29allocator<16G_DabingMorphObj>>Fv |
| 0x0029d310 | 92 | 13 | std::vector<16G_DabingMorphObj,Q23std29allocator<16G_DabingMorphObj>> | {dtor} | __dt__Q23std64vector<16G_DabingMorphObj,Q23std29allocator<16G_DabingMorphObj>>Fv |
| 0x0029d370 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std29allocator<16G_DabingMorphObj>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details63compressed_pair_imp<Q23std29allocator<16G_DabingMorphObj>,Ui,1>FUi |
| 0x0029d380 | 488 | GLOBAL | G_IndicatorsClass | {ctor} | __ct__17G_IndicatorsClassFv |
| 0x0029d570 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,Ui,1>FUi |
| 0x0029d580 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,Ui,1>FUi |
| 0x0029d590 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std38allocator<25G_MIIntroSubtitlesWaiting>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details72compressed_pair_imp<Q23std38allocator<25G_MIIntroSubtitlesWaiting>,Ui,1>FUi |
| 0x0029d5a0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18G_MIIntroSubtitles>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18G_MIIntroSubtitles>,Ui,1>FUi |
| 0x0029d5b0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18G_MISubtitleStruct>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18G_MISubtitleStruct>,Ui,1>FUi |
| 0x0029d5c0 | 552 | GLOBAL | G_IndicatorsClass | ResetValues | ResetValues__17G_IndicatorsClassFv |
| 0x0029d7f0 | 424 | GLOBAL | G_IndicatorsClass | LoadFont | LoadFont__17G_IndicatorsClassFbUi |
| 0x0029d9a0 | 112 | GLOBAL | G_IndicatorsClass | LoadFontFake | LoadFontFake__17G_IndicatorsClassFUiUi |
| 0x0029da10 | 212 | GLOBAL | G_IndicatorsClass | LanguageSelect | LanguageSelect__17G_IndicatorsClassFUi |
| 0x0029daf0 | 876 | GLOBAL | G_IndicatorsClass | Create | Create__17G_IndicatorsClassFv |
| 0x0029de60 | 384 | GLOBAL | G_IndicatorsClass | Destroy | Destroy__17G_IndicatorsClassFv |
| 0x0029dfe0 | 156 | GLOBAL | G_IndicatorsClass | NahrajMapu | NahrajMapu__17G_IndicatorsClassFv |
| 0x0029e080 | 92 | GLOBAL | G_IndicatorsClass | UvolniMapu | UvolniMapu__17G_IndicatorsClassFb |
| 0x0029e0e0 | 6316 | GLOBAL | G_IndicatorsClass | GameInit | GameInit__17G_IndicatorsClassFv |
| 0x0029f990 | 412 | GLOBAL | G_IndicatorsClass | GameDone | GameDone__17G_IndicatorsClassFb |
| 0x0029fb30 | 32 | GLOBAL | G_IndicatorsClass | TimerSetTime | TimerSetTime__17G_IndicatorsClassFUiUiUi |
| 0x0029fb50 | 308 | GLOBAL | G_IndicatorsClass | TimerSetInterval | TimerSetInterval__17G_IndicatorsClassFUi |
| 0x0029fc90 | 28 | GLOBAL | G_IndicatorsClass | TimerGetTime | TimerGetTime__17G_IndicatorsClassFPUiPUiPUi |
| 0x0029fcb0 | 40 | GLOBAL | G_IndicatorsClass | TimerGetInterval | TimerGetInterval__17G_IndicatorsClassFPUi |
| 0x0029fce0 | 1204 | GLOBAL | G_IndicatorsClass | _SpocitejRafiku2 | _SpocitejRafiku2__17G_IndicatorsClassFP12G_RafickaDefP14G_RafickaValueR9S_vector2P8TLVERTEX |
| 0x002a01a0 | 408 | GLOBAL | G_IndicatorsClass | _PridejWingPodklad | _PridejWingPodklad__17G_IndicatorsClassFP8TLVERTEXPUiUiUi |
| 0x002a0340 | 260 | GLOBAL | G_IndicatorsClass | _PridejAmmoPodklad | _PridejAmmoPodklad__17G_IndicatorsClassFP8TLVERTEXPUi |
| 0x002a0450 | 1080 | GLOBAL | G_IndicatorsClass | _PridejWingLives | _PridejWingLives__17G_IndicatorsClassFP8TLVERTEXPUiUiUi |
| 0x002a0890 | 2152 | GLOBAL | G_IndicatorsClass | _PridejAmmo | _PridejAmmo__17G_IndicatorsClassFP8TLVERTEXPUiUiUi |
| 0x002a1100 | 36 | GLOBAL | G_IndicatorsClass | TachometrSetGear | TachometrSetGear__17G_IndicatorsClassFib |
| 0x002a1130 | 880 | GLOBAL | G_IndicatorsClass | TachometrSetValues | TachometrSetValues__17G_IndicatorsClassFffff |
| 0x002a14a0 | 24 | GLOBAL | G_IndicatorsClass | PlayerSetLives | PlayerSetLives__17G_IndicatorsClassFUi |
| 0x002a14c0 | 32 | GLOBAL | G_IndicatorsClass | PlayerSetAmmo | PlayerSetAmmo__17G_IndicatorsClassFUiUi |
| 0x002a14e0 | 8 | GLOBAL | G_IndicatorsClass | PlayerSetAccuracy | PlayerSetAccuracy__17G_IndicatorsClassFf |
| 0x002a14f0 | 40 | GLOBAL | G_IndicatorsClass | PlayerSetWingmanName | PlayerSetWingmanName__17G_IndicatorsClassFUiUi |
| 0x002a1520 | 36 | GLOBAL | G_IndicatorsClass | PlayerSetWingmanLives | PlayerSetWingmanLives__17G_IndicatorsClassFUiUi |
| 0x002a1550 | 240 | GLOBAL |  | _SetTransparency__FP9I3D_modelff | _SetTransparency__FP9I3D_modelff |
| 0x002a1640 | 172 | GLOBAL | G_IndicatorsClass | DrawInventory | DrawInventory__17G_IndicatorsClassFv |
| 0x002a16f0 | 1348 | GLOBAL | G_IndicatorsClass | InventoryScroll | InventoryScroll__17G_IndicatorsClassFP11G_InventoryUiUii |
| 0x002a1c40 | 5664 | GLOBAL | G_IndicatorsClass | _InventorySet | _InventorySet__17G_IndicatorsClassFiiiii |
| 0x002a3260 | 8 | GLOBAL | G_IndicatorsClass | SubtitlesClear | SubtitlesClear__17G_IndicatorsClassFv |
| 0x002a3270 | 8 | GLOBAL | G_IndicatorsClass | SubtitlesAdd | SubtitlesAdd__17G_IndicatorsClassFUiUib |
| 0x002a3280 | 876 | GLOBAL | G_IndicatorsClass | IntroSubtitlesAddIntern | IntroSubtitlesAddIntern__17G_IndicatorsClassFUif |
| 0x002a35f0 | 152 | GLOBAL | G_IndicatorsClass | IntroSubtitlesAdd | IntroSubtitlesAdd__17G_IndicatorsClassFUiUi |
| 0x002a3690 | 172 | GLOBAL | G_IndicatorsClass | IntroSubtitlesAddSimple | IntroSubtitlesAddSimple__17G_IndicatorsClassFUi |
| 0x002a3740 | 48 | GLOBAL | G_IndicatorsClass | IntroSubtitlesClear | IntroSubtitlesClear__17G_IndicatorsClassFv |
| 0x002a3770 | 8 | GLOBAL | G_IndicatorsClass | MissionHelpClear | MissionHelpClear__17G_IndicatorsClassFv |
| 0x002a3780 | 420 | GLOBAL | G_IndicatorsClass | MissionHelpAdd | MissionHelpAdd__17G_IndicatorsClassFUi |
| 0x002a3930 | 52 | GLOBAL | G_IndicatorsClass | MissionHelpSetTime | MissionHelpSetTime__17G_IndicatorsClassFUi |
| 0x002a3970 | 436 | GLOBAL | G_IndicatorsClass | TutorialAddText | TutorialAddText__17G_IndicatorsClassFUi |
| 0x002a3b30 | 260 | 13 | gif::vector<Ui,8> | reserve | reserve__Q23gif12vector<Ui,8>Fi |
| 0x002a3c40 | 56 | GLOBAL | G_IndicatorsClass | MapEnable | MapEnable__17G_IndicatorsClassFb |
| 0x002a3c80 | 8 | GLOBAL | G_IndicatorsClass | MapIsEnabled | MapIsEnabled__17G_IndicatorsClassFv |
| 0x002a3c90 | 12 | GLOBAL | G_IndicatorsClass | ConsoleClear | ConsoleClear__17G_IndicatorsClassFv |
| 0x002a3ca0 | 196 | GLOBAL | G_IndicatorsClass | ConsoleAddText | ConsoleAddText__17G_IndicatorsClassFPUcUi |
| 0x002a3d70 | 176 | GLOBAL | G_IndicatorsClass | ConsoleAddTextByID | ConsoleAddTextByID__17G_IndicatorsClassFUiUi |
| 0x002a3e20 | 8 | GLOBAL | G_IndicatorsClass | CompassSetDestination | CompassSetDestination__17G_IndicatorsClassFP9I3D_frame |
| 0x002a3e30 | 8 | GLOBAL | G_IndicatorsClass | CompassSetDestination2 | CompassSetDestination2__17G_IndicatorsClassFP9I3D_frame |
| 0x002a3e40 | 8 | GLOBAL | G_IndicatorsClass | ThiefIndicatorSetValue | ThiefIndicatorSetValue__17G_IndicatorsClassFf |
| 0x002a3e50 | 8 | GLOBAL | G_IndicatorsClass | RefullingIndicatorSetValue | RefullingIndicatorSetValue__17G_IndicatorsClassFf |
| 0x002a3e60 | 1008 | GLOBAL | G_IndicatorsClass | WideScreenSetRatio | WideScreenSetRatio__17G_IndicatorsClassFf |
| 0x002a4250 | 96 | GLOBAL | G_IndicatorsClass | FadeInOut | FadeInOut__17G_IndicatorsClassFbUiUi |
| 0x002a42b0 | 48 | GLOBAL | G_IndicatorsClass | FadeInOutGetState | FadeInOutGetState__17G_IndicatorsClassFv |
| 0x002a42e0 | 20 | GLOBAL | G_IndicatorsClass | FadeInOutIsDark | FadeInOutIsDark__17G_IndicatorsClassFv |
| 0x002a4300 | 48 | GLOBAL | G_IndicatorsClass | RadarClear | RadarClear__17G_IndicatorsClassFv |
| 0x002a4330 | 8 | GLOBAL | G_IndicatorsClass | RadarSetPlayerCar | RadarSetPlayerCar__17G_IndicatorsClassFP5C_car |
| 0x002a4340 | 200 | GLOBAL | G_IndicatorsClass | RadarAddCar | RadarAddCar__17G_IndicatorsClassFP5C_carUi |
| 0x002a4410 | 92 | GLOBAL | G_IndicatorsClass | RadarRemoveCar | RadarRemoveCar__17G_IndicatorsClassFP5C_car |
| 0x002a4470 | 132 | GLOBAL | G_IndicatorsClass | RadarAddObject | RadarAddObject__17G_IndicatorsClassFP18tDynamicCollObjectUi |
| 0x002a4500 | 92 | GLOBAL | G_IndicatorsClass | RadarRemoveObject | RadarRemoveObject__17G_IndicatorsClassFP18tDynamicCollObject |
| 0x002a4560 | 728 | GLOBAL | G_IndicatorsClass | RadarSetLeftEdge | RadarSetLeftEdge__17G_IndicatorsClassFPQ23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>> |
| 0x002a4840 | 728 | GLOBAL | G_IndicatorsClass | RadarSetRightEdge | RadarSetRightEdge__17G_IndicatorsClassFPQ23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>> |
| 0x002a4b20 | 112 | GLOBAL | G_IndicatorsClass | RaceSetStartFlag | RaceSetStartFlag__17G_IndicatorsClassFUi |
| 0x002a4b90 | 132 | GLOBAL | G_IndicatorsClass | RaceSetTime | RaceSetTime__17G_IndicatorsClassFUi |
| 0x002a4c20 | 132 | GLOBAL | G_IndicatorsClass | RaceSetLapTime | RaceSetLapTime__17G_IndicatorsClassFUi |
| 0x002a4cb0 | 132 | GLOBAL | G_IndicatorsClass | RaceSetBestTime | RaceSetBestTime__17G_IndicatorsClassFUi |
| 0x002a4d40 | 92 | GLOBAL | G_IndicatorsClass | RaceSetPosition | RaceSetPosition__17G_IndicatorsClassFUiUi |
| 0x002a4da0 | 100 | GLOBAL | G_IndicatorsClass | RaceSetLaps | RaceSetLaps__17G_IndicatorsClassFUiUi |
| 0x002a4e10 | 68 | GLOBAL | G_IndicatorsClass | RaceFlashText | RaceFlashText__17G_IndicatorsClassFPUcf |
| 0x002a4e60 | 104 | GLOBAL | G_IndicatorsClass | BonusSetCount | BonusSetCount__17G_IndicatorsClassFi |
| 0x002a4ed0 | 26044 | GLOBAL | G_IndicatorsClass | DrawAll | DrawAll__17G_IndicatorsClassFv |
| 0x002ab490 | 508 | GLOBAL | G_IndicatorsClass | MapGetScrollPos | MapGetScrollPos__17G_IndicatorsClassFRfRff |
| 0x002ab690 | 7840 | GLOBAL | G_IndicatorsClass | DrawMap | DrawMap__17G_IndicatorsClassFfff |
| 0x002ad530 | 16 | 13 | S_vector2 | {ctor} | __ct__9S_vector2Fff |
| 0x002ad540 | 476 | GLOBAL | G_IndicatorsClass | DrawMessage | DrawMessage__17G_IndicatorsClassFPUc |
| 0x002ad720 | 8736 | GLOBAL | G_IndicatorsClass | ResolutionChanged | ResolutionChanged__17G_IndicatorsClassFv |
| 0x002af940 | 60 | GLOBAL | G_IndicatorsClass | AByloTicho | AByloTicho__17G_IndicatorsClassFi |
| 0x002af980 | 8428 | GLOBAL | G_IndicatorsClass | Tick | Tick__17G_IndicatorsClassFUi |
| 0x002b1a70 | 8 | GLOBAL |  | _GIconReset__Fv | _GIconReset__Fv |
| 0x002b1a80 | 2408 | GLOBAL |  | _GIconAdd__FRfffUiUc | _GIconAdd__FRfffUiUc |
| 0x002b23f0 | 116 | GLOBAL |  | _GIconDraw__Fv | _GIconDraw__Fv |
| 0x002b2470 | 196 | GLOBAL | G_IndicatorsClass | TextSizeMultiLine | TextSizeMultiLine__17G_IndicatorsClassFRfRfPUcfUi |
| 0x002b2540 | 292 | GLOBAL | G_IndicatorsClass | TextSize | TextSize__17G_IndicatorsClassFPUcfUiPUc |
| 0x002b2670 | 352 | GLOBAL | G_IndicatorsClass | HledejZlom | HledejZlom__17G_IndicatorsClassFPUcffUiPUc |
| 0x002b27d0 | 2608 | GLOBAL | G_IndicatorsClass | OutText | OutText__17G_IndicatorsClassFPUcffffUiUiUiPUc |
| 0x002b3200 | 212 | GLOBAL | G_IndicatorsClass | OutTextMultiLine | OutTextMultiLine__17G_IndicatorsClassFPUcffffUiUiUi |
| 0x002b32e0 | 760 | GLOBAL | G_IndicatorsClass | ParheliaSetFov | ParheliaSetFov__17G_IndicatorsClassFP10I3D_cameraf |
| 0x002b35e0 | 1212 | GLOBAL | G_IndicatorsClass | PleaseWait | PleaseWait__17G_IndicatorsClassFb |
| 0x002b3aa0 | 92 | GLOBAL |  | GameCd_Callback__FUi | GameCd_Callback__FUi |
| 0x002b3b00 | 132 | WEAK | std::__vector_imp<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>,0> | erase | erase__Q23std76__vector_imp<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>,0>FP18G_MISubtitleStruct |
| 0x002b3b90 | 1180 | WEAK | std::__vector_imp<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>,0> | insert | insert__Q23std76__vector_imp<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>,0>FP18G_MISubtitleStructUiRC18G_MISubtitleStruct |
| 0x002b4030 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18G_MISubtitleStruct>,Ui,1> | swap | swap__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18G_MISubtitleStruct>,Ui,1>FRQ310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18G_MISubtitleStruct>,Ui,1> |
| 0x002b4050 | 368 | WEAK | std::__vector_imp<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>,0> | reserve | reserve__Q23std76__vector_imp<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>,0>FUi |
| 0x002b41c0 | 208 | WEAK | std::__vector_imp<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>,0> | erase | erase__Q23std76__vector_imp<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>,0>FP18G_MIIntroSubtitles |
| 0x002b4290 | 64 | 13 | G_MIIntroSubtitles | operator= | __as__18G_MIIntroSubtitlesFRC18G_MIIntroSubtitles |
| 0x002b42d0 | 1388 | WEAK | std::__vector_imp<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>,0> | insert | insert__Q23std76__vector_imp<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>,0>FP18G_MIIntroSubtitlesUiRC18G_MIIntroSubtitles |
| 0x002b4840 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std31allocator<18G_MIIntroSubtitles>,Ui,1> | swap | swap__Q310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18G_MIIntroSubtitles>,Ui,1>FRQ310Metrowerks7details65compressed_pair_imp<Q23std31allocator<18G_MIIntroSubtitles>,Ui,1> |
| 0x002b4860 | 416 | WEAK | std::__vector_imp<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>,0> | reserve | reserve__Q23std76__vector_imp<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>,0>FUi |
| 0x002b4a00 | 116 | WEAK | std::__vector_imp<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>,0> | erase | erase__Q23std90__vector_imp<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>,0>FP25G_MIIntroSubtitlesWaiting |
| 0x002b4a80 | 1052 | WEAK | std::__vector_imp<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>,0> | insert | insert__Q23std90__vector_imp<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>,0>FP25G_MIIntroSubtitlesWaitingUiRC25G_MIIntroSubtitlesWaiting |
| 0x002b4ea0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std38allocator<25G_MIIntroSubtitlesWaiting>,Ui,1> | swap | swap__Q310Metrowerks7details72compressed_pair_imp<Q23std38allocator<25G_MIIntroSubtitlesWaiting>,Ui,1>FRQ310Metrowerks7details72compressed_pair_imp<Q23std38allocator<25G_MIIntroSubtitlesWaiting>,Ui,1> |
| 0x002b4ec0 | 352 | WEAK | std::__vector_imp<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>,0> | reserve | reserve__Q23std90__vector_imp<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>,0>FUi |
| 0x002b5020 | 132 | WEAK | std::__vector_imp<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,0> | erase | erase__Q23std102__vector_imp<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,0>FPQ217G_IndicatorsClass10G_RadarCar |
| 0x002b50b0 | 1180 | WEAK | std::__vector_imp<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,0> | insert | insert__Q23std102__vector_imp<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,0>FPQ217G_IndicatorsClass10G_RadarCarUiRCQ217G_IndicatorsClass10G_RadarCar |
| 0x002b5550 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,Ui,1> | swap | swap__Q310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,Ui,1>FRQ310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,Ui,1> |
| 0x002b5570 | 368 | WEAK | std::__vector_imp<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,0> | reserve | reserve__Q23std102__vector_imp<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>,0>FUi |
| 0x002b56e0 | 116 | WEAK | std::__vector_imp<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,0> | erase | erase__Q23std104__vector_imp<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,0>FPQ217G_IndicatorsClass11G_RadarItem |
| 0x002b5760 | 1052 | WEAK | std::__vector_imp<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,0> | insert | insert__Q23std104__vector_imp<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,0>FPQ217G_IndicatorsClass11G_RadarItemUiRCQ217G_IndicatorsClass11G_RadarItem |
| 0x002b5b80 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,Ui,1> | swap | swap__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,Ui,1>FRQ310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,Ui,1> |
| 0x002b5ba0 | 352 | WEAK | std::__vector_imp<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,0> | reserve | reserve__Q23std104__vector_imp<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>,0>FUi |
| 0x002b5d00 | 72 | WEAK | std::__vector_deleter<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>> | clear | clear__Q23std78__vector_deleter<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>>Fv |
| 0x002b5d50 | 104 | WEAK | std::__vector_deleter<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>> | {dtor} | __dt__Q23std78__vector_deleter<18G_MISubtitleStruct,Q23std31allocator<18G_MISubtitleStruct>>Fv |
| 0x002b5dc0 | 80 | WEAK | std::__vector_deleter<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>> | clear | clear__Q23std78__vector_deleter<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>>Fv |
| 0x002b5e10 | 104 | WEAK | std::__vector_deleter<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>> | {dtor} | __dt__Q23std78__vector_deleter<18G_MIIntroSubtitles,Q23std31allocator<18G_MIIntroSubtitles>>Fv |
| 0x002b5e80 | 72 | WEAK | std::__vector_deleter<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>> | clear | clear__Q23std92__vector_deleter<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>>Fv |
| 0x002b5ed0 | 104 | WEAK | std::__vector_deleter<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>> | {dtor} | __dt__Q23std92__vector_deleter<25G_MIIntroSubtitlesWaiting,Q23std38allocator<25G_MIIntroSubtitlesWaiting>>Fv |
| 0x002b5f40 | 72 | WEAK | std::__vector_deleter<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>> | clear | clear__Q23std104__vector_deleter<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>>Fv |
| 0x002b5f90 | 104 | WEAK | std::__vector_deleter<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>> | {dtor} | __dt__Q23std104__vector_deleter<Q217G_IndicatorsClass10G_RadarCar,Q23std44allocator<Q217G_IndicatorsClass10G_RadarCar>>Fv |
| 0x002b6000 | 72 | WEAK | std::__vector_deleter<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>> | clear | clear__Q23std106__vector_deleter<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>>Fv |
| 0x002b6050 | 104 | WEAK | std::__vector_deleter<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>> | {dtor} | __dt__Q23std106__vector_deleter<Q217G_IndicatorsClass11G_RadarItem,Q23std45allocator<Q217G_IndicatorsClass11G_RadarItem>>Fv |
| 0x002b60c0 | 272 | 13 | G_IndicatorsClass | {dtor} | __dt__17G_IndicatorsClassFv |
| 0x002b61d0 | 1252 | GLOBAL | G_LoadSaveClass | Init | Init__15G_LoadSaveClassFv |
| 0x002b66c0 | 88 | GLOBAL | G_LoadSaveClass | Close | Close__15G_LoadSaveClassFv |
| 0x002b6720 | 676 | GLOBAL | G_LoadSaveClass | ProfileChange | ProfileChange__15G_LoadSaveClassFv |
| 0x002b69d0 | 440 | GLOBAL | G_LoadSaveClass | {ctor} | __ct__15G_LoadSaveClassFv |
| 0x002b6b90 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std32allocator<19G_SaveGame_FileHead>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details66compressed_pair_imp<Q23std32allocator<19G_SaveGame_FileHead>,Ui,1>FUi |
| 0x002b6ba0 | 224 | GLOBAL | G_LoadSaveClass | AllDefault | AllDefault__15G_LoadSaveClassFv |
| 0x002b6c80 | 300 | GLOBAL | G_LoadSaveClass | ProfileEnumFiles | ProfileEnumFiles__15G_LoadSaveClassFv |
| 0x002b6db0 | 44 | GLOBAL | G_LoadSaveClass | ProfileEnumRelease | ProfileEnumRelease__15G_LoadSaveClassFv |
| 0x002b6de0 | 8 | GLOBAL | G_LoadSaveClass | ProfileGetList | ProfileGetList__15G_LoadSaveClassFv |
| 0x002b6df0 | 104 | GLOBAL | G_LoadSaveClass | ProfileSelect | ProfileSelect__15G_LoadSaveClassFUi |
| 0x002b6e60 | 152 | GLOBAL | G_LoadSaveClass | ProfileOverwrite | ProfileOverwrite__15G_LoadSaveClassFUi |
| 0x002b6f00 | 580 | GLOBAL | G_LoadSaveClass | ProfileCreate | ProfileCreate__15G_LoadSaveClassFPUc |
| 0x002b7150 | 420 | GLOBAL | G_LoadSaveClass | ProfileDelete | ProfileDelete__15G_LoadSaveClassFUi |
| 0x002b7300 | 388 | GLOBAL | G_LoadSaveClass | ProfileLoad | ProfileLoad__15G_LoadSaveClassFv |
| 0x002b7490 | 352 | GLOBAL | G_LoadSaveClass | ProfileSave | ProfileSave__15G_LoadSaveClassFv |
| 0x002b75f0 | 208 | GLOBAL | G_LoadSaveClass | SaveGameEnumAddFile | SaveGameEnumAddFile__15G_LoadSaveClassFP19G_SaveGame_FileHead |
| 0x002b76c0 | 664 | GLOBAL | G_LoadSaveClass | SaveGameEnumFiles | SaveGameEnumFiles__15G_LoadSaveClassFb |
| 0x002b7960 | 8 | GLOBAL | G_LoadSaveClass | SaveGameEnumRelease | SaveGameEnumRelease__15G_LoadSaveClassFv |
| 0x002b7970 | 96 | GLOBAL | G_LoadSaveClass | SaveGameSelect | SaveGameSelect__15G_LoadSaveClassFUi |
| 0x002b79d0 | 8 | GLOBAL | G_LoadSaveClass | SaveGameGetSelectedNumber | SaveGameGetSelectedNumber__15G_LoadSaveClassFv |
| 0x002b79e0 | 416 | GLOBAL | G_LoadSaveClass | SaveGameCreate | SaveGameCreate__15G_LoadSaveClassFUib |
| 0x002b7b80 | 40 | GLOBAL | G_LoadSaveClass | SaveGameNewGame | SaveGameNewGame__15G_LoadSaveClassFv |
| 0x002b7bb0 | 92 | GLOBAL | G_LoadSaveClass | SaveGameLoadHeader | SaveGameLoadHeader__15G_LoadSaveClassFUiP19G_SaveGame_FileHead |
| 0x002b7c10 | 3672 | GLOBAL | G_LoadSaveClass | SaveGameLoad | SaveGameLoad__15G_LoadSaveClassFv |
| 0x002b8a70 | 96 | 13 | GM_LoadProgress | {dtor} | __dt__15GM_LoadProgressFv |
| 0x002b8ad0 | 356 | GLOBAL | G_LoadSaveClass | _SaveProc | _SaveProc__15G_LoadSaveClassFP8G_StreamP7C_actorP20G_SaveGame_ActorHeadUiRUi |
| 0x002b8c40 | 1456 | GLOBAL | G_LoadSaveClass | SaveGameSave | SaveGameSave__15G_LoadSaveClassFb |
| 0x002b91f0 | 8 | GLOBAL | G_LoadSaveClass | GetScreenShotIdx | GetScreenShotIdx__15G_LoadSaveClassFUi |
| 0x002b9200 | 380 | GLOBAL | G_LoadSaveClass | PSXFullSaveProfileOK | PSXFullSaveProfileOK__15G_LoadSaveClassFv |
| 0x002b9380 | 3056 | GLOBAL | G_LoadSaveClass | SaveGameFromScript | SaveGameFromScript__15G_LoadSaveClassFUibb |
| 0x002b9f70 | 132 | GLOBAL | G_LoadSaveClass | SaveGameSetLastActive | SaveGameSetLastActive__15G_LoadSaveClassFv |
| 0x002ba000 | 356 | GLOBAL | G_LoadSaveClass | SaveGameSetRetryActive | SaveGameSetRetryActive__15G_LoadSaveClassFv |
| 0x002ba170 | 128 | GLOBAL | G_LoadSaveClass | LanguageSelect | LanguageSelect__15G_LoadSaveClassFUi |
| 0x002ba1f0 | 236 | GLOBAL | G_EnabledCars | HaveCar | HaveCar__13G_EnabledCarsFUib |
| 0x002ba2e0 | 100 | GLOBAL | G_CarDatabase | {ctor} | __ct__13G_CarDatabaseFv |
| 0x002ba350 | 364 | GLOBAL | G_CarDatabase | Init | Init__13G_CarDatabaseFv |
| 0x002ba4c0 | 88 | GLOBAL | G_CarDatabase | Close | Close__13G_CarDatabaseFv |
| 0x002ba520 | 72 | GLOBAL | G_CarDatabase | GetCar | GetCar__13G_CarDatabaseFUi |
| 0x002ba570 | 76 | GLOBAL | G_CarDatabase | GetCarDetails | GetCarDetails__13G_CarDatabaseFUi |
| 0x002ba5c0 | 72 | GLOBAL | G_CarDatabase | GetPlayerModel | GetPlayerModel__13G_CarDatabaseFUi |
| 0x002ba610 | 68 | GLOBAL | G_CarDatabase | GetCarName | GetCarName__13G_CarDatabaseFUi |
| 0x002ba660 | 304 | GLOBAL | G_CarDatabase | GetCarSecondName | GetCarSecondName__13G_CarDatabaseFUiUi |
| 0x002ba790 | 128 | GLOBAL | G_CarDatabase | RepairIndex | RepairIndex__13G_CarDatabaseFRUi |
| 0x002ba810 | 104 | GLOBAL | G_CarDatabase | GetCarMaxColor | GetCarMaxColor__13G_CarDatabaseFUi |
| 0x002ba880 | 192 | GLOBAL | G_CarDatabase | GetCarModel | GetCarModel__13G_CarDatabaseFUiPUcb |
| 0x002ba940 | 164 | GLOBAL | G_CarDatabase | GetCarParams | GetCarParams__13G_CarDatabaseFUiPUiPUiPUi |
| 0x002ba9f0 | 44 | GLOBAL | G_CarDatabase | GetIDByParams | GetIDByParams__13G_CarDatabaseFUiUiUi |
| 0x002baa20 | 132 | GLOBAL | G_CarDatabase | GetIdxByModel | GetIdxByModel__13G_CarDatabaseFPUc |
| 0x002baab0 | 276 | GLOBAL | G_CarDatabase | GetIdxByModelLong | GetIdxByModelLong__13G_CarDatabaseFPUcRUi |
| 0x002babd0 | 148 | GLOBAL | G_CarDatabase | GetIDByActor | GetIDByActor__13G_CarDatabaseFP5C_car |
| 0x002bac70 | 288 | GLOBAL | G_CarDatabase | GetWreckageByModel | GetWreckageByModel__13G_CarDatabaseFPUc |
| 0x002bad90 | 68 | GLOBAL | G_CarDatabase | GetWreckageByID | GetWreckageByID__13G_CarDatabaseFi |
| 0x002bade0 | 288 | GLOBAL | G_CarDatabase | GetShadowByModel | GetShadowByModel__13G_CarDatabaseFPUc |
| 0x002baf00 | 164 | GLOBAL | G_CarDatabase | GetCivilMaskByModel | GetCivilMaskByModel__13G_CarDatabaseFPUc |
| 0x002bafb0 | 232 | GLOBAL | G_CarDatabase | GetCarsByClass | GetCarsByClass__13G_CarDatabaseFUiPQ23std32vector<Ui,Q23std13allocator<Ui>>P13G_EnabledCars |
| 0x002bb0a0 | 180 | GLOBAL | G_CarDatabase | GetCarsByRaceMask | GetCarsByRaceMask__13G_CarDatabaseFUiPQ23std32vector<Ui,Q23std13allocator<Ui>> |
| 0x002bb160 | 232 | GLOBAL | G_CarDatabase | GetCarsByManufacturer | GetCarsByManufacturer__13G_CarDatabaseFUiPQ23std32vector<Ui,Q23std13allocator<Ui>>P13G_EnabledCars |
| 0x002bb250 | 484 | GLOBAL | G_CarDatabase | GetCarsByCarcyclopediaMask | GetCarsByCarcyclopediaMask__13G_CarDatabaseFUiPQ23gif12vector<Ui,8> |
| 0x002bb440 | 84 | GLOBAL | G_CarDatabase | EnableCarsForFree | EnableCarsForFree__13G_CarDatabaseFUi |
| 0x002bb4a0 | 80 | GLOBAL | G_CarDatabase | IsEnabledForRace | IsEnabledForRace__13G_CarDatabaseFUiUi |
| 0x002bb4f0 | 80 | GLOBAL | G_CarDatabase | IsEnabledForChampionship | IsEnabledForChampionship__13G_CarDatabaseFUiUi |
| 0x002bb540 | 80 | GLOBAL | G_CarDatabase | IsEnabledForFreeride | IsEnabledForFreeride__13G_CarDatabaseFUiUi |
| 0x002bb590 | 76 | GLOBAL | G_CarDatabase | GetCarClass | GetCarClass__13G_CarDatabaseFUi |
| 0x002bb5e0 | 164 | GLOBAL | G_CarDatabase | GetStealTime | GetStealTime__13G_CarDatabaseFPUc |
| 0x002bb690 | 156 | GLOBAL | G_CarDatabase | GetPlayerCivilMaskByModel | GetPlayerCivilMaskByModel__13G_CarDatabaseFPUc |
| 0x002bb730 | 212 | GLOBAL | G_CarDatabase | LoadDetails | LoadDetails__13G_CarDatabaseFv |
| 0x002bb810 | 68 | GLOBAL | G_CarDatabase | UnloadDetails | UnloadDetails__13G_CarDatabaseFv |
| 0x002bb860 | 64 | GLOBAL | G_Stream | {ctor} | __ct__8G_StreamFv |
| 0x002bb8a0 | 84 | GLOBAL | G_Stream | {dtor} | __dt__8G_StreamFv |
| 0x002bb900 | 16 | GLOBAL | G_Stream | SetParams | SetParams__8G_StreamFUibb |
| 0x002bb910 | 8 | GLOBAL | G_Stream | SetFlags | SetFlags__8G_StreamFUi |
| 0x002bb920 | 184 | GLOBAL | G_Stream | OpenRead | OpenRead__8G_StreamFPc |
| 0x002bb9e0 | 320 | GLOBAL | G_Stream | OpenWrite | OpenWrite__8G_StreamFPc |
| 0x002bbb20 | 128 | GLOBAL | G_Stream | Close | Close__8G_StreamFv |
| 0x002bbba0 | 208 | GLOBAL | G_Stream | WantWrite | WantWrite__8G_StreamFUi |
| 0x002bbc70 | 80 | GLOBAL | G_Stream | Write | Write__8G_StreamFv |
| 0x002bbcc0 | 220 | GLOBAL | G_Stream | Read | Read__8G_StreamFUi |
| 0x002bbda0 | 8 | GLOBAL | G_Stream | Crypt | Crypt__8G_StreamFPvUi |
| 0x002bbdb0 | 8 | GLOBAL | G_Stream | Decrypt | Decrypt__8G_StreamFPvUi |
| 0x002bbdc0 | 364 | GLOBAL | G_Stream | WriteAdr | WriteAdr__8G_StreamFPvUi |
| 0x002bbf30 | 260 | GLOBAL | G_Stream | WriteAdrAtPos | WriteAdrAtPos__8G_StreamFPvUiUi |
| 0x002bc040 | 308 | GLOBAL | G_Stream | ReadAdr | ReadAdr__8G_StreamFPvUi |
| 0x002bc180 | 140 | GLOBAL | G_LoadSaveClass | ReplayTick | ReplayTick__15G_LoadSaveClassFRi |
| 0x002bc210 | 60 | GLOBAL | G_LoadSaveClass | ReplaySet | ReplaySet__15G_LoadSaveClassFbb |
| 0x002bc250 | 232 | GLOBAL | G_LoadSaveClass | ReplayLoad | ReplayLoad__15G_LoadSaveClassFUi |
| 0x002bc340 | 84 | GLOBAL | G_LoadSaveClass | _replayGetNext | _replayGetNext__15G_LoadSaveClassFv |
| 0x002bc3a0 | 68 | GLOBAL | G_LoadSaveClass | _replayPutNext | _replayPutNext__15G_LoadSaveClassFUi |
| 0x002bc3f0 | 16 | GLOBAL | G_LoadSaveClass | RaceCarsEnable | RaceCarsEnable__15G_LoadSaveClassFUi |
| 0x002bc400 | 20 | GLOBAL | G_LoadSaveClass | RaceCarsDisable | RaceCarsDisable__15G_LoadSaveClassFUi |
| 0x002bc420 | 8 | GLOBAL | G_LoadSaveClass | RaceCarsGetEnabled | RaceCarsGetEnabled__15G_LoadSaveClassFv |
| 0x002bc430 | 104 | GLOBAL | G_LoadSaveClass | RaceCarsAddWin | RaceCarsAddWin__15G_LoadSaveClassFUi |
| 0x002bc4a0 | 8 | GLOBAL | G_LoadSaveClass | ObjectivesClear | ObjectivesClear__15G_LoadSaveClassFv |
| 0x002bc4b0 | 160 | GLOBAL | G_LoadSaveClass | ObjectivesAdd | ObjectivesAdd__15G_LoadSaveClassFUi |
| 0x002bc550 | 140 | GLOBAL | G_LoadSaveClass | ObjectivesRemove | ObjectivesRemove__15G_LoadSaveClassFUi |
| 0x002bc5e0 | 8 | GLOBAL | G_LoadSaveClass | ObjectivesComplete | ObjectivesComplete__15G_LoadSaveClassFUi |
| 0x002bc5f0 | 124 | GLOBAL | G_LoadSaveClass | ObjectivesDisplay | ObjectivesDisplay__15G_LoadSaveClassFv |
| 0x002bc670 | 8 | GLOBAL | G_LoadSaveClass | ObjectivesGetList | ObjectivesGetList__15G_LoadSaveClassFv |
| 0x002bc680 | 176 | GLOBAL | G_LoadSaveClass | FreerideEnableCar | FreerideEnableCar__15G_LoadSaveClassFUc |
| 0x002bc730 | 8 | GLOBAL | G_LoadSaveClass | FreerideGetEnabledCars | FreerideGetEnabledCars__15G_LoadSaveClassFv |
| 0x002bc740 | 316 | GLOBAL | G_LoadSaveClass | Refresh | Refresh__15G_LoadSaveClassFUi |
| 0x002bc880 | 1260 | GLOBAL |  | RenderOneCredit__FP15G_CreditsStructf | RenderOneCredit__FP15G_CreditsStructf |
| 0x002bcd70 | 320 | GLOBAL |  | ParseCredits__Fv | ParseCredits__Fv |
| 0x002bceb0 | 1104 | GLOBAL | G_LoadSaveClass | ExecuteCredits | ExecuteCredits__15G_LoadSaveClassFv |
| 0x002bd300 | 8 | GLOBAL | G_LoadSaveClass | FreerideEnable | FreerideEnable__15G_LoadSaveClassFUi |
| 0x002bd310 | 44 | GLOBAL | G_LoadSaveClass | FreerideStoreA | FreerideStoreA__15G_LoadSaveClassFUiUifff |
| 0x002bd340 | 12 | GLOBAL | G_LoadSaveClass | FreerideStoreB | FreerideStoreB__15G_LoadSaveClassFUi |
| 0x002bd350 | 12 | GLOBAL | G_LoadSaveClass | FreerideGetState | FreerideGetState__15G_LoadSaveClassFv |
| 0x002bd360 | 40 | GLOBAL | G_LoadSaveClass | FreerideSetupFromScript | FreerideSetupFromScript__15G_LoadSaveClassFv |
| 0x002bd390 | 1780 | GLOBAL | G_LoadSaveClass | FreerideSetupCar | FreerideSetupCar__15G_LoadSaveClassFv |
| 0x002bda90 | 1760 | GLOBAL | G_LoadSaveClass | FreerideSetupPlayer | FreerideSetupPlayer__15G_LoadSaveClassFv |
| 0x002be170 | 56 | GLOBAL | G_LoadSaveClass | FreerideSetup | FreerideSetup__15G_LoadSaveClassFv |
| 0x002be1b0 | 540 | GLOBAL | G_LoadSaveClass | FreerideSetupAfter | FreerideSetupAfter__15G_LoadSaveClassFv |
| 0x002be3d0 | 68 | GLOBAL | G_LoadSaveClass | DefaultSettings | DefaultSettings__15G_LoadSaveClassFUi |
| 0x002be420 | 56 | GLOBAL | G_LoadSaveClass | PSXGetSaveBase | PSXGetSaveBase__15G_LoadSaveClassFUi |
| 0x002be460 | 108 | GLOBAL | G_LoadSaveClass | PSXGetSaveIndex | PSXGetSaveIndex__15G_LoadSaveClassFUi |
| 0x002be4d0 | 32 | GLOBAL | G_LoadSaveClass | PSXGetSaveNum | PSXGetSaveNum__15G_LoadSaveClassFUi |
| 0x002be4f0 | 72 | GLOBAL | G_LoadSaveClass | PSXGetSaveType | PSXGetSaveType__15G_LoadSaveClassFUi |
| 0x002be540 | 40 | GLOBAL | G_LoadSaveClass | PSXIsSaveFull | PSXIsSaveFull__15G_LoadSaveClassFUi |
| 0x002be570 | 144 | GLOBAL | G_LoadSaveClass | PSXGetMem | PSXGetMem__15G_LoadSaveClassFUi |
| 0x002be600 | 100 | GLOBAL | G_LoadSaveClass | PSXReallocFullSaveMem | PSXReallocFullSaveMem__15G_LoadSaveClassFUi |
| 0x002be670 | 60 | GLOBAL | G_LoadSaveClass | PSXFreeFullSaveMem | PSXFreeFullSaveMem__15G_LoadSaveClassFv |
| 0x002be6b0 | 56 | GLOBAL | G_LoadSaveClass | PSXEnableSave | PSXEnableSave__15G_LoadSaveClassFUi |
| 0x002be6f0 | 24 | GLOBAL | G_LoadSaveClass | PSXIsEnabledSave | PSXIsEnabledSave__15G_LoadSaveClassFUi |
| 0x002be710 | 28 | GLOBAL | G_LoadSaveClass | PSXGetSelectedProfileSystemName | PSXGetSelectedProfileSystemName__15G_LoadSaveClassFv |
| 0x002be730 | 48 | GLOBAL | G_LoadSaveClass | PSXGetProfileSystemName | PSXGetProfileSystemName__15G_LoadSaveClassFUi |
| 0x002be760 | 44 | GLOBAL | G_LoadSaveClass | PSXGetSaveMem | PSXGetSaveMem__15G_LoadSaveClassFUi |
| 0x002be790 | 192 | GLOBAL | G_LoadSaveClass | PSXEnumFullSave | PSXEnumFullSave__15G_LoadSaveClassFv |
| 0x002be850 | 84 | GLOBAL | G_LoadSaveClass | PSXEnumFreeFullSave | PSXEnumFreeFullSave__15G_LoadSaveClassFv |
| 0x002be8b0 | 72 | GLOBAL | G_LoadSaveClass | PSXFilterSave | PSXFilterSave__15G_LoadSaveClassFUi |
| 0x002be900 | 300 | GLOBAL | G_LoadSaveClass | PSXEnumFullSaveFinish | PSXEnumFullSaveFinish__15G_LoadSaveClassFv |
| 0x002bea30 | 196 | GLOBAL | G_LoadSaveClass | PSXGetFullSaveName | PSXGetFullSaveName__15G_LoadSaveClassFUi |
| 0x002beb00 | 200 | GLOBAL | G_LoadSaveClass | PSXFullSaveExistsCard | PSXFullSaveExistsCard__15G_LoadSaveClassFUiPUi |
| 0x002bebd0 | 44 | GLOBAL | G_LoadSaveClass | PSXGetFullSaveBrowserSize | PSXGetFullSaveBrowserSize__15G_LoadSaveClassFUi |
| 0x002bec00 | 76 | GLOBAL | G_LoadSaveClass | PSXIsInSameSaveGroup | PSXIsInSameSaveGroup__15G_LoadSaveClassFUiUi |
| 0x002bec50 | 96 | GLOBAL | G_LoadSaveClass | PSXFreeFSheaderaname | PSXFreeFSheaderaname__15G_LoadSaveClassFv |
| 0x002becb0 | 284 | GLOBAL | G_LoadSaveClass | PSXProfileExists | PSXProfileExists__15G_LoadSaveClassFPcUi |
| 0x002bedd0 | 52 | GLOBAL | G_LoadSaveClass | PSXIEUVER | PSXIEUVER__15G_LoadSaveClassFv |
| 0x002bee10 | 200 | GLOBAL | G_LoadSaveClass | PSXSelectedProfileCard | PSXSelectedProfileCard__15G_LoadSaveClassFUiP14MEMORYCARDINFO |
| 0x002beee0 | 132 | GLOBAL | G_LoadSaveClass | PSXProfileExistsOnMoreCards | PSXProfileExistsOnMoreCards__15G_LoadSaveClassFUi |
| 0x002bef70 | 76 | GLOBAL | G_LoadSaveClass | PSXGetFullSaveName | PSXGetFullSaveName__15G_LoadSaveClassFP19G_SaveGame_FileHead |
| 0x002befc0 | 312 | GLOBAL | G_LoadSaveClass | PSXSelectOKSave | PSXSelectOKSave__15G_LoadSaveClassFv |
| 0x002bf100 | 204 | GLOBAL | G_LoadSaveClass | PSXProfileCheckProfileVersion | PSXProfileCheckProfileVersion__15G_LoadSaveClassFv |
| 0x002bf1d0 | 8 | GLOBAL | G_LoadSaveClass | PSXProfileBrowserSize | PSXProfileBrowserSize__15G_LoadSaveClassFv |
| 0x002bf1e0 | 76 | GLOBAL | G_LoadSaveClass | PSXProfileGetNumCorrupt | PSXProfileGetNumCorrupt__15G_LoadSaveClassFv |
| 0x002bf230 | 176 | GLOBAL | G_LoadSaveClass | PSXProfileSelect | PSXProfileSelect__15G_LoadSaveClassFUi |
| 0x002bf2e0 | 132 | GLOBAL | G_LoadSaveClass | PSXProfileSelectLastCorrupt | PSXProfileSelectLastCorrupt__15G_LoadSaveClassFv |
| 0x002bf370 | 152 | GLOBAL | G_LoadSaveClass | PSXProfileGetCorruptName | PSXProfileGetCorruptName__15G_LoadSaveClassFv |
| 0x002bf410 | 164 | GLOBAL | G_LoadSaveClass | PSXLoadProductIdFromDTA | PSXLoadProductIdFromDTA__15G_LoadSaveClassFv |
| 0x002bf4c0 | 144 | GLOBAL | G_LoadSaveClass | ProfileNumEnabledSaves | ProfileNumEnabledSaves__15G_LoadSaveClassFv |
| 0x002bf550 | 1252 | WEAK | std::__vector_imp<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>,0> | insert | insert__Q23std78__vector_imp<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>,0>FP19G_SaveGame_FileHeadUiRC19G_SaveGame_FileHead |
| 0x002bfa40 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std32allocator<19G_SaveGame_FileHead>,Ui,1> | swap | swap__Q310Metrowerks7details66compressed_pair_imp<Q23std32allocator<19G_SaveGame_FileHead>,Ui,1>FRQ310Metrowerks7details66compressed_pair_imp<Q23std32allocator<19G_SaveGame_FileHead>,Ui,1> |
| 0x002bfa60 | 120 | 13 | G_SaveGame_FileHead | operator= | __as__19G_SaveGame_FileHeadFRC19G_SaveGame_FileHead |
| 0x002bfae0 | 80 | WEAK | std::__vector_deleter<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>> | clear | clear__Q23std80__vector_deleter<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>>Fv |
| 0x002bfb30 | 104 | WEAK | std::__vector_deleter<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>> | {dtor} | __dt__Q23std80__vector_deleter<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>>Fv |
| 0x002bfba0 | 84 | 13 | G_CarDatabase | {dtor} | __dt__13G_CarDatabaseFv |
| 0x002bfc00 | 196 | 13 | G_LoadSaveClass | {dtor} | __dt__15G_LoadSaveClassFv |
| 0x002bfcd0 | 160 | GLOBAL | G_CarcCache | Clear | Clear__11G_CarcCacheFv |
| 0x002bfd70 | 164 | GLOBAL | G_CarcCache | RemoveFirst | RemoveFirst__11G_CarcCacheFi |
| 0x002bfe20 | 68 | GLOBAL | G_CarcCache | LimitSize | LimitSize__11G_CarcCacheFv |
| 0x002bfe70 | 164 | GLOBAL | G_CarcCache | HaveModel | HaveModel__11G_CarcCacheFPc |
| 0x002bff20 | 416 | GLOBAL | G_CarcCache | GetModel | GetModel__11G_CarcCacheFPc |
| 0x002c00c0 | 260 | 13 | gif::vector<P9I3D_model,8> | reserve | reserve__Q23gif21vector<P9I3D_model,8>Fi |
| 0x002c01d0 | 120 | GLOBAL | G_CarcCache | GetModelById | GetModelById__11G_CarcCacheFUiUi |
| 0x002c0250 | 528 | GLOBAL | G_CarcCache | CacheModelByIdx | CacheModelByIdx__11G_CarcCacheFUiib |
| 0x002c0460 | 312 | GLOBAL | G_CarcCache | CacheModelByColor | CacheModelByColor__11G_CarcCacheFUii |
| 0x002c05a0 | 1348 | GLOBAL | GM_Carcyclopedia | OnCreate | OnCreate__16GM_CarcyclopediaFv |
| 0x002c0af0 | 260 | 13 | gif::vector<PUc,8> | reserve | reserve__Q23gif13vector<PUc,8>Fi |
| 0x002c0c00 | 100 | GLOBAL | GM_Carcyclopedia | OnClose | OnClose__16GM_CarcyclopediaFv |
| 0x002c0c70 | 1296 | GLOBAL | GM_Carcyclopedia | OnTick | OnTick__16GM_CarcyclopediaFUi |
| 0x002c1180 | 768 | GLOBAL | GM_Carcyclopedia | OnKey | OnKey__16GM_CarcyclopediaFUcUi |
| 0x002c1480 | 120 | GLOBAL | GM_Carcyclopedia | OnClick | OnClick__16GM_CarcyclopediaFUi |
| 0x002c1500 | 428 | GLOBAL | GM_Carcyclopedia | OnMouse | OnMouse__16GM_CarcyclopediaFiiiUiUi |
| 0x002c16b0 | 100 | 13 | GM_Component | IsOverActive | IsOverActive__12GM_ComponentFff |
| 0x002c1720 | 864 | GLOBAL | GM_Carcyclopedia | RefreshCar | RefreshCar__16GM_CarcyclopediaFiib |
| 0x002c1a80 | 184 | 13 | GM_Carcyclopedia | {dtor} | __dt__16GM_CarcyclopediaFv |
| 0x002c1b40 | 12 | 13 | GM_Carcyclopedia | OnBeforeCreate | OnBeforeCreate__16GM_CarcyclopediaFv |
| 0x002c1b50 | 16 | 13 | GM_Menu | OnBeforeExchange | OnBeforeExchange__7GM_MenuFv |
| 0x002c1b60 | 16 | 13 | GM_Menu | OnExchange | OnExchange__7GM_MenuFv |
| 0x002c1b70 | 8 | 13 | GM_Menu | OnCreate | OnCreate__7GM_MenuFv |
| 0x002c1b80 | 8 | 13 | GM_Menu | OnCreateChild | OnCreateChild__7GM_MenuFP7GM_MenuUi |
| 0x002c1b90 | 8 | 13 | GM_Menu | OnCloseChild | OnCloseChild__7GM_MenuFP7GM_MenuUiUi |
| 0x002c1ba0 | 8 | 13 | GM_Menu | OnChange | OnChange__7GM_MenuFP12GM_ComponentP12GM_Component |
| 0x002c1bb0 | 8 | 13 | GM_Menu | MartinovaKlavesa | MartinovaKlavesa__7GM_MenuFUiUi |
| 0x002c1bc0 | 116 | 13 | G_CarcCache | {dtor} | __dt__11G_CarcCacheFv |
| 0x002c1c40 | 72 | 13 | G_Factory<12GM_Component> | Make | Make__25G_Factory<12GM_Component>FP15GM_DesignStructP12GM_Component |
| 0x002c1c90 | 56 | 13 | G_Factory<12GM_Component> | MakeDup | MakeDup__25G_Factory<12GM_Component>FP12GM_Component |
| 0x002c1cd0 | 72 | 13 | G_Factory<10GM_Nahrada> | Make | Make__23G_Factory<10GM_Nahrada>FP15GM_DesignStructP12GM_Component |
| 0x002c1d20 | 56 | 13 | G_Factory<10GM_Nahrada> | MakeDup | MakeDup__23G_Factory<10GM_Nahrada>FP12GM_Component |
| 0x002c1d60 | 72 | 13 | G_Factory<7GM_UpUp> | Make | Make__19G_Factory<7GM_UpUp>FP15GM_DesignStructP12GM_Component |
| 0x002c1db0 | 56 | 13 | G_Factory<7GM_UpUp> | MakeDup | MakeDup__19G_Factory<7GM_UpUp>FP12GM_Component |
| 0x002c1df0 | 72 | 13 | G_Factory<9GM_Window> | Make | Make__21G_Factory<9GM_Window>FP15GM_DesignStructP12GM_Component |
| 0x002c1e40 | 56 | 13 | G_Factory<9GM_Window> | MakeDup | MakeDup__21G_Factory<9GM_Window>FP12GM_Component |
| 0x002c1e80 | 72 | 13 | G_Factory<14GM_WindowClean> | Make | Make__27G_Factory<14GM_WindowClean>FP15GM_DesignStructP12GM_Component |
| 0x002c1ed0 | 56 | 13 | G_Factory<14GM_WindowClean> | MakeDup | MakeDup__27G_Factory<14GM_WindowClean>FP12GM_Component |
| 0x002c1f10 | 72 | 13 | G_Factory<17GM_WindowTutorial> | Make | Make__30G_Factory<17GM_WindowTutorial>FP15GM_DesignStructP12GM_Component |
| 0x002c1f60 | 56 | 13 | G_Factory<17GM_WindowTutorial> | MakeDup | MakeDup__30G_Factory<17GM_WindowTutorial>FP12GM_Component |
| 0x002c1fa0 | 72 | 13 | G_Factory<15GM_WindowScroll> | Make | Make__28G_Factory<15GM_WindowScroll>FP15GM_DesignStructP12GM_Component |
| 0x002c1ff0 | 56 | 13 | G_Factory<15GM_WindowScroll> | MakeDup | MakeDup__28G_Factory<15GM_WindowScroll>FP12GM_Component |
| 0x002c2030 | 72 | 13 | G_Factory<10GM_PS2Help> | Make | Make__23G_Factory<10GM_PS2Help>FP15GM_DesignStructP12GM_Component |
| 0x002c2080 | 56 | 13 | G_Factory<10GM_PS2Help> | MakeDup | MakeDup__23G_Factory<10GM_PS2Help>FP12GM_Component |
| 0x002c20c0 | 72 | 13 | G_Factory<11GM_PS2Help2> | Make | Make__24G_Factory<11GM_PS2Help2>FP15GM_DesignStructP12GM_Component |
| 0x002c2110 | 56 | 13 | G_Factory<11GM_PS2Help2> | MakeDup | MakeDup__24G_Factory<11GM_PS2Help2>FP12GM_Component |
| 0x002c2150 | 72 | 13 | G_Factory<7GM_Mapa> | Make | Make__19G_Factory<7GM_Mapa>FP15GM_DesignStructP12GM_Component |
| 0x002c21a0 | 56 | 13 | G_Factory<7GM_Mapa> | MakeDup | MakeDup__19G_Factory<7GM_Mapa>FP12GM_Component |
| 0x002c21e0 | 72 | 13 | G_Factory<13GM_StaticText> | Make | Make__26G_Factory<13GM_StaticText>FP15GM_DesignStructP12GM_Component |
| 0x002c2230 | 56 | 13 | G_Factory<13GM_StaticText> | MakeDup | MakeDup__26G_Factory<13GM_StaticText>FP12GM_Component |
| 0x002c2270 | 72 | 13 | G_Factory<16GM_StaticTextFit> | Make | Make__29G_Factory<16GM_StaticTextFit>FP15GM_DesignStructP12GM_Component |
| 0x002c22c0 | 56 | 13 | G_Factory<16GM_StaticTextFit> | MakeDup | MakeDup__29G_Factory<16GM_StaticTextFit>FP12GM_Component |
| 0x002c2300 | 72 | 13 | G_Factory<16GM_StaticTextMem> | Make | Make__29G_Factory<16GM_StaticTextMem>FP15GM_DesignStructP12GM_Component |
| 0x002c2350 | 56 | 13 | G_Factory<16GM_StaticTextMem> | MakeDup | MakeDup__29G_Factory<16GM_StaticTextMem>FP12GM_Component |
| 0x002c2390 | 72 | 13 | G_Factory<17GM_StaticTextMem2> | Make | Make__30G_Factory<17GM_StaticTextMem2>FP15GM_DesignStructP12GM_Component |
| 0x002c23e0 | 56 | 13 | G_Factory<17GM_StaticTextMem2> | MakeDup | MakeDup__30G_Factory<17GM_StaticTextMem2>FP12GM_Component |
| 0x002c2420 | 72 | 13 | G_Factory<16GM_StaticTextNum> | Make | Make__29G_Factory<16GM_StaticTextNum>FP15GM_DesignStructP12GM_Component |
| 0x002c2470 | 56 | 13 | G_Factory<16GM_StaticTextNum> | MakeDup | MakeDup__29G_Factory<16GM_StaticTextNum>FP12GM_Component |
| 0x002c24b0 | 72 | 13 | G_Factory<17GM_StaticTextTime> | Make | Make__30G_Factory<17GM_StaticTextTime>FP15GM_DesignStructP12GM_Component |
| 0x002c2500 | 56 | 13 | G_Factory<17GM_StaticTextTime> | MakeDup | MakeDup__30G_Factory<17GM_StaticTextTime>FP12GM_Component |
| 0x002c2540 | 72 | 13 | G_Factory<7GM_Line> | Make | Make__19G_Factory<7GM_Line>FP15GM_DesignStructP12GM_Component |
| 0x002c2590 | 56 | 13 | G_Factory<7GM_Line> | MakeDup | MakeDup__19G_Factory<7GM_Line>FP12GM_Component |
| 0x002c25d0 | 72 | 13 | G_Factory<12GM_Rectangle> | Make | Make__25G_Factory<12GM_Rectangle>FP15GM_DesignStructP12GM_Component |
| 0x002c2620 | 56 | 13 | G_Factory<12GM_Rectangle> | MakeDup | MakeDup__25G_Factory<12GM_Rectangle>FP12GM_Component |
| 0x002c2660 | 72 | 13 | G_Factory<13GM_RectangleB> | Make | Make__26G_Factory<13GM_RectangleB>FP15GM_DesignStructP12GM_Component |
| 0x002c26b0 | 56 | 13 | G_Factory<13GM_RectangleB> | MakeDup | MakeDup__26G_Factory<13GM_RectangleB>FP12GM_Component |
| 0x002c26f0 | 72 | 13 | G_Factory<11GM_Triangle> | Make | Make__24G_Factory<11GM_Triangle>FP15GM_DesignStructP12GM_Component |
| 0x002c2740 | 56 | 13 | G_Factory<11GM_Triangle> | MakeDup | MakeDup__24G_Factory<11GM_Triangle>FP12GM_Component |
| 0x002c2780 | 72 | 13 | G_Factory<11GM_Orezavac> | Make | Make__24G_Factory<11GM_Orezavac>FP15GM_DesignStructP12GM_Component |
| 0x002c27d0 | 56 | 13 | G_Factory<11GM_Orezavac> | MakeDup | MakeDup__24G_Factory<11GM_Orezavac>FP12GM_Component |
| 0x002c2810 | 72 | 13 | G_Factory<19GM_RectangleTexture> | Make | Make__32G_Factory<19GM_RectangleTexture>FP15GM_DesignStructP12GM_Component |
| 0x002c2860 | 56 | 13 | G_Factory<19GM_RectangleTexture> | MakeDup | MakeDup__32G_Factory<19GM_RectangleTexture>FP12GM_Component |
| 0x002c28a0 | 72 | 13 | G_Factory<14GM_ProgressBar> | Make | Make__27G_Factory<14GM_ProgressBar>FP15GM_DesignStructP12GM_Component |
| 0x002c28f0 | 56 | 13 | G_Factory<14GM_ProgressBar> | MakeDup | MakeDup__27G_Factory<14GM_ProgressBar>FP12GM_Component |
| 0x002c2930 | 72 | 13 | G_Factory<11GM_MenuItem> | Make | Make__24G_Factory<11GM_MenuItem>FP15GM_DesignStructP12GM_Component |
| 0x002c2980 | 56 | 13 | G_Factory<11GM_MenuItem> | MakeDup | MakeDup__24G_Factory<11GM_MenuItem>FP12GM_Component |
| 0x002c29c0 | 72 | 13 | G_Factory<14GM_MenuItemMem> | Make | Make__27G_Factory<14GM_MenuItemMem>FP15GM_DesignStructP12GM_Component |
| 0x002c2a10 | 56 | 13 | G_Factory<14GM_MenuItemMem> | MakeDup | MakeDup__27G_Factory<14GM_MenuItemMem>FP12GM_Component |
| 0x002c2a50 | 72 | 13 | G_Factory<14GM_MenuItemNum> | Make | Make__27G_Factory<14GM_MenuItemNum>FP15GM_DesignStructP12GM_Component |
| 0x002c2aa0 | 56 | 13 | G_Factory<14GM_MenuItemNum> | MakeDup | MakeDup__27G_Factory<14GM_MenuItemNum>FP12GM_Component |
| 0x002c2ae0 | 72 | 13 | G_Factory<16GM_RaceRoundItem> | Make | Make__29G_Factory<16GM_RaceRoundItem>FP15GM_DesignStructP12GM_Component |
| 0x002c2b30 | 56 | 13 | G_Factory<16GM_RaceRoundItem> | MakeDup | MakeDup__29G_Factory<16GM_RaceRoundItem>FP12GM_Component |
| 0x002c2b70 | 72 | 13 | G_Factory<9GM_Button> | Make | Make__21G_Factory<9GM_Button>FP15GM_DesignStructP12GM_Component |
| 0x002c2bc0 | 56 | 13 | G_Factory<9GM_Button> | MakeDup | MakeDup__21G_Factory<9GM_Button>FP12GM_Component |
| 0x002c2c00 | 72 | 13 | G_Factory<15GM_ButtonShrink> | Make | Make__28G_Factory<15GM_ButtonShrink>FP15GM_DesignStructP12GM_Component |
| 0x002c2c50 | 56 | 13 | G_Factory<15GM_ButtonShrink> | MakeDup | MakeDup__28G_Factory<15GM_ButtonShrink>FP12GM_Component |
| 0x002c2c90 | 72 | 13 | G_Factory<14GM_ButtonClean> | Make | Make__27G_Factory<14GM_ButtonClean>FP15GM_DesignStructP12GM_Component |
| 0x002c2ce0 | 56 | 13 | G_Factory<14GM_ButtonClean> | MakeDup | MakeDup__27G_Factory<14GM_ButtonClean>FP12GM_Component |
| 0x002c2d20 | 72 | 13 | G_Factory<9GM_Slider> | Make | Make__21G_Factory<9GM_Slider>FP15GM_DesignStructP12GM_Component |
| 0x002c2d70 | 56 | 13 | G_Factory<9GM_Slider> | MakeDup | MakeDup__21G_Factory<9GM_Slider>FP12GM_Component |
| 0x002c2db0 | 72 | 13 | G_Factory<12GM_SliderNum> | Make | Make__25G_Factory<12GM_SliderNum>FP15GM_DesignStructP12GM_Component |
| 0x002c2e00 | 56 | 13 | G_Factory<12GM_SliderNum> | MakeDup | MakeDup__25G_Factory<12GM_SliderNum>FP12GM_Component |
| 0x002c2e40 | 72 | 13 | G_Factory<16GM_SliderInteger> | Make | Make__29G_Factory<16GM_SliderInteger>FP15GM_DesignStructP12GM_Component |
| 0x002c2e90 | 56 | 13 | G_Factory<16GM_SliderInteger> | MakeDup | MakeDup__29G_Factory<16GM_SliderInteger>FP12GM_Component |
| 0x002c2ed0 | 72 | 13 | G_Factory<12GM_NetPlayer> | Make | Make__25G_Factory<12GM_NetPlayer>FP15GM_DesignStructP12GM_Component |
| 0x002c2f20 | 56 | 13 | G_Factory<12GM_NetPlayer> | MakeDup | MakeDup__25G_Factory<12GM_NetPlayer>FP12GM_Component |
| 0x002c2f60 | 72 | 13 | G_Factory<11GM_TextEdit> | Make | Make__24G_Factory<11GM_TextEdit>FP15GM_DesignStructP12GM_Component |
| 0x002c2fb0 | 56 | 13 | G_Factory<11GM_TextEdit> | MakeDup | MakeDup__24G_Factory<11GM_TextEdit>FP12GM_Component |
| 0x002c2ff0 | 72 | 13 | G_Factory<11GM_CheckBox> | Make | Make__24G_Factory<11GM_CheckBox>FP15GM_DesignStructP12GM_Component |
| 0x002c3040 | 56 | 13 | G_Factory<11GM_CheckBox> | MakeDup | MakeDup__24G_Factory<11GM_CheckBox>FP12GM_Component |
| 0x002c3080 | 72 | 13 | G_Factory<15GM_ButtonSelect> | Make | Make__28G_Factory<15GM_ButtonSelect>FP15GM_DesignStructP12GM_Component |
| 0x002c30d0 | 56 | 13 | G_Factory<15GM_ButtonSelect> | MakeDup | MakeDup__28G_Factory<15GM_ButtonSelect>FP12GM_Component |
| 0x002c3110 | 72 | 13 | G_Factory<19GM_ButtonSelectBool> | Make | Make__32G_Factory<19GM_ButtonSelectBool>FP15GM_DesignStructP12GM_Component |
| 0x002c3160 | 56 | 13 | G_Factory<19GM_ButtonSelectBool> | MakeDup | MakeDup__32G_Factory<19GM_ButtonSelectBool>FP12GM_Component |
| 0x002c31a0 | 72 | 13 | G_Factory<10GM_EditNum> | Make | Make__23G_Factory<10GM_EditNum>FP15GM_DesignStructP12GM_Component |
| 0x002c31f0 | 56 | 13 | G_Factory<10GM_EditNum> | MakeDup | MakeDup__23G_Factory<10GM_EditNum>FP12GM_Component |
| 0x002c3230 | 72 | 13 | G_Factory<14GM_PSXTextEdit> | Make | Make__27G_Factory<14GM_PSXTextEdit>FP15GM_DesignStructP12GM_Component |
| 0x002c3280 | 56 | 13 | G_Factory<14GM_PSXTextEdit> | MakeDup | MakeDup__27G_Factory<14GM_PSXTextEdit>FP12GM_Component |
| 0x002c32c0 | 52 | 13 | GM_PSXTextEdit | {ctor} | __ct__14GM_PSXTextEditFP12GM_Component |
| 0x002c3300 | 96 | 13 | GM_TextEdit | {dtor} | __dt__11GM_TextEditFv |
| 0x002c3360 | 100 | 13 | GM_Component | {dtor} | __dt__12GM_ComponentFv |
| 0x002c33d0 | 104 | 13 | gif::vector<P12GM_Component,8> | {dtor} | __dt__Q23gif25vector<P12GM_Component,8>Fv |
| 0x002c3440 | 8 | 13 |  | _aligned_free__FPv | _aligned_free__FPv |
| 0x002c3450 | 52 | 13 | GM_PSXTextEdit | {ctor} | __ct__14GM_PSXTextEditFP15GM_DesignStructP12GM_Component |
| 0x002c3490 | 104 | 13 | GM_EditNum | {ctor} | __ct__10GM_EditNumFP12GM_Component |
| 0x002c3500 | 120 | 13 | GM_EditNum | {ctor} | __ct__10GM_EditNumFP15GM_DesignStructP12GM_Component |
| 0x002c3580 | 52 | 13 | GM_ButtonSelectBool | {ctor} | __ct__19GM_ButtonSelectBoolFP12GM_Component |
| 0x002c35c0 | 68 | 13 | GM_ButtonSelectBool | {ctor} | __ct__19GM_ButtonSelectBoolFP15GM_DesignStructP12GM_Component |
| 0x002c3610 | 52 | 13 | GM_ButtonSelect | {ctor} | __ct__15GM_ButtonSelectFP12GM_Component |
| 0x002c3650 | 64 | 13 | GM_ButtonSelect | {ctor} | __ct__15GM_ButtonSelectFP15GM_DesignStructP12GM_Component |
| 0x002c3690 | 72 | 13 | GM_CheckBox | {ctor} | __ct__11GM_CheckBoxFP12GM_Component |
| 0x002c36e0 | 68 | 13 | GM_CheckBox | {ctor} | __ct__11GM_CheckBoxFP15GM_DesignStructP12GM_Component |
| 0x002c3730 | 96 | 13 | GM_TextEdit | {ctor} | __ct__11GM_TextEditFP12GM_Component |
| 0x002c3790 | 112 | 13 | GM_TextEdit | {ctor} | __ct__11GM_TextEditFP15GM_DesignStructP12GM_Component |
| 0x002c3800 | 52 | 13 | GM_NetPlayer | {ctor} | __ct__12GM_NetPlayerFP12GM_Component |
| 0x002c3840 | 172 | 13 | GM_NetPlayer | {ctor} | __ct__12GM_NetPlayerFP15GM_DesignStructP12GM_Component |
| 0x002c38f0 | 88 | 13 | GM_SliderInteger | {ctor} | __ct__16GM_SliderIntegerFP12GM_Component |
| 0x002c3950 | 80 | 13 | GM_SliderInteger | {ctor} | __ct__16GM_SliderIntegerFP15GM_DesignStructP12GM_Component |
| 0x002c39a0 | 72 | 13 | GM_SliderNum | {ctor} | __ct__12GM_SliderNumFP12GM_Component |
| 0x002c39f0 | 72 | 13 | GM_SliderNum | {ctor} | __ct__12GM_SliderNumFP15GM_DesignStructP12GM_Component |
| 0x002c3a40 | 72 | 13 | GM_Slider | {ctor} | __ct__9GM_SliderFP12GM_Component |
| 0x002c3a90 | 72 | 13 | GM_Slider | {ctor} | __ct__9GM_SliderFP15GM_DesignStructP12GM_Component |
| 0x002c3ae0 | 52 | 13 | GM_ButtonClean | {ctor} | __ct__14GM_ButtonCleanFP12GM_Component |
| 0x002c3b20 | 64 | 13 | GM_ButtonClean | {ctor} | __ct__14GM_ButtonCleanFP15GM_DesignStructP12GM_Component |
| 0x002c3b60 | 52 | 13 | GM_ButtonShrink | {ctor} | __ct__15GM_ButtonShrinkFP12GM_Component |
| 0x002c3ba0 | 64 | 13 | GM_ButtonShrink | {ctor} | __ct__15GM_ButtonShrinkFP15GM_DesignStructP12GM_Component |
| 0x002c3be0 | 52 | 13 | GM_Button | {ctor} | __ct__9GM_ButtonFP12GM_Component |
| 0x002c3c20 | 64 | 13 | GM_Button | {ctor} | __ct__9GM_ButtonFP15GM_DesignStructP12GM_Component |
| 0x002c3c60 | 100 | 13 | GM_RaceRoundItem | {ctor} | __ct__16GM_RaceRoundItemFP12GM_Component |
| 0x002c3cd0 | 132 | 13 | GM_RaceRoundItem | {ctor} | __ct__16GM_RaceRoundItemFP15GM_DesignStructP12GM_Component |
| 0x002c3d60 | 100 | 13 | GM_MenuItemNum | {ctor} | __ct__14GM_MenuItemNumFP12GM_Component |
| 0x002c3dd0 | 116 | 13 | GM_MenuItemNum | {ctor} | __ct__14GM_MenuItemNumFP15GM_DesignStructP12GM_Component |
| 0x002c3e50 | 52 | 13 | GM_MenuItemMem | {ctor} | __ct__14GM_MenuItemMemFP12GM_Component |
| 0x002c3e90 | 64 | 13 | GM_MenuItemMem | {ctor} | __ct__14GM_MenuItemMemFP15GM_DesignStructP12GM_Component |
| 0x002c3ed0 | 52 | 13 | GM_MenuItem | {ctor} | __ct__11GM_MenuItemFP12GM_Component |
| 0x002c3f10 | 64 | 13 | GM_MenuItem | {ctor} | __ct__11GM_MenuItemFP15GM_DesignStructP12GM_Component |
| 0x002c3f50 | 56 | 13 | GM_ProgressBar | {ctor} | __ct__14GM_ProgressBarFP12GM_Component |
| 0x002c3f90 | 56 | 13 | GM_ProgressBar | {ctor} | __ct__14GM_ProgressBarFP15GM_DesignStructP12GM_Component |
| 0x002c3fd0 | 56 | 13 | GM_RectangleTexture | {ctor} | __ct__19GM_RectangleTextureFP12GM_Component |
| 0x002c4010 | 56 | 13 | GM_RectangleTexture | {ctor} | __ct__19GM_RectangleTextureFP15GM_DesignStructP12GM_Component |
| 0x002c4050 | 52 | 13 | GM_Orezavac | {ctor} | __ct__11GM_OrezavacFP12GM_Component |
| 0x002c4090 | 52 | 13 | GM_Orezavac | {ctor} | __ct__11GM_OrezavacFP15GM_DesignStructP12GM_Component |
| 0x002c40d0 | 52 | 13 | GM_Triangle | {ctor} | __ct__11GM_TriangleFP12GM_Component |
| 0x002c4110 | 52 | 13 | GM_Triangle | {ctor} | __ct__11GM_TriangleFP15GM_DesignStructP12GM_Component |
| 0x002c4150 | 52 | 13 | GM_RectangleB | {ctor} | __ct__13GM_RectangleBFP12GM_Component |
| 0x002c4190 | 52 | 13 | GM_RectangleB | {ctor} | __ct__13GM_RectangleBFP15GM_DesignStructP12GM_Component |
| 0x002c41d0 | 52 | 13 | GM_Rectangle | {ctor} | __ct__12GM_RectangleFP12GM_Component |
| 0x002c4210 | 52 | 13 | GM_Rectangle | {ctor} | __ct__12GM_RectangleFP15GM_DesignStructP12GM_Component |
| 0x002c4250 | 52 | 13 | GM_Line | {ctor} | __ct__7GM_LineFP12GM_Component |
| 0x002c4290 | 52 | 13 | GM_Line | {ctor} | __ct__7GM_LineFP15GM_DesignStructP12GM_Component |
| 0x002c42d0 | 56 | 13 | GM_StaticTextTime | {ctor} | __ct__17GM_StaticTextTimeFP12GM_Component |
| 0x002c4310 | 56 | 13 | GM_StaticTextTime | {ctor} | __ct__17GM_StaticTextTimeFP15GM_DesignStructP12GM_Component |
| 0x002c4350 | 56 | 13 | GM_StaticTextNum | {ctor} | __ct__16GM_StaticTextNumFP12GM_Component |
| 0x002c4390 | 56 | 13 | GM_StaticTextNum | {ctor} | __ct__16GM_StaticTextNumFP15GM_DesignStructP12GM_Component |
| 0x002c43d0 | 52 | 13 | GM_StaticTextMem2 | {ctor} | __ct__17GM_StaticTextMem2FP12GM_Component |
| 0x002c4410 | 52 | 13 | GM_StaticTextMem2 | {ctor} | __ct__17GM_StaticTextMem2FP15GM_DesignStructP12GM_Component |
| 0x002c4450 | 52 | 13 | GM_StaticTextMem | {ctor} | __ct__16GM_StaticTextMemFP12GM_Component |
| 0x002c4490 | 52 | 13 | GM_StaticTextMem | {ctor} | __ct__16GM_StaticTextMemFP15GM_DesignStructP12GM_Component |
| 0x002c44d0 | 52 | 13 | GM_StaticTextFit | {ctor} | __ct__16GM_StaticTextFitFP12GM_Component |
| 0x002c4510 | 52 | 13 | GM_StaticTextFit | {ctor} | __ct__16GM_StaticTextFitFP15GM_DesignStructP12GM_Component |
| 0x002c4550 | 52 | 13 | GM_StaticText | {ctor} | __ct__13GM_StaticTextFP12GM_Component |
| 0x002c4590 | 52 | 13 | GM_StaticText | {ctor} | __ct__13GM_StaticTextFP15GM_DesignStructP12GM_Component |
| 0x002c45d0 | 52 | 13 | GM_Mapa | {ctor} | __ct__7GM_MapaFP12GM_Component |
| 0x002c4610 | 52 | 13 | GM_Mapa | {ctor} | __ct__7GM_MapaFP15GM_DesignStructP12GM_Component |
| 0x002c4650 | 52 | 13 | GM_PS2Help2 | {ctor} | __ct__11GM_PS2Help2FP12GM_Component |
| 0x002c4690 | 52 | 13 | GM_PS2Help2 | {ctor} | __ct__11GM_PS2Help2FP15GM_DesignStructP12GM_Component |
| 0x002c46d0 | 52 | 13 | GM_PS2Help | {ctor} | __ct__10GM_PS2HelpFP12GM_Component |
| 0x002c4710 | 52 | 13 | GM_PS2Help | {ctor} | __ct__10GM_PS2HelpFP15GM_DesignStructP12GM_Component |
| 0x002c4750 | 88 | 13 | GM_WindowScroll | {ctor} | __ct__15GM_WindowScrollFP12GM_Component |
| 0x002c47b0 | 88 | 13 | GM_WindowScroll | {ctor} | __ct__15GM_WindowScrollFP15GM_DesignStructP12GM_Component |
| 0x002c4810 | 52 | 13 | GM_WindowTutorial | {ctor} | __ct__17GM_WindowTutorialFP12GM_Component |
| 0x002c4850 | 52 | 13 | GM_WindowTutorial | {ctor} | __ct__17GM_WindowTutorialFP15GM_DesignStructP12GM_Component |
| 0x002c4890 | 52 | 13 | GM_WindowClean | {ctor} | __ct__14GM_WindowCleanFP12GM_Component |
| 0x002c48d0 | 52 | 13 | GM_WindowClean | {ctor} | __ct__14GM_WindowCleanFP15GM_DesignStructP12GM_Component |
| 0x002c4910 | 52 | 13 | GM_Window | {ctor} | __ct__9GM_WindowFP12GM_Component |
| 0x002c4950 | 52 | 13 | GM_Window | {ctor} | __ct__9GM_WindowFP15GM_DesignStructP12GM_Component |
| 0x002c4990 | 52 | 13 | GM_UpUp | {ctor} | __ct__7GM_UpUpFP12GM_Component |
| 0x002c49d0 | 52 | 13 | GM_UpUp | {ctor} | __ct__7GM_UpUpFP15GM_DesignStructP12GM_Component |
| 0x002c4a10 | 52 | 13 | GM_Nahrada | {ctor} | __ct__10GM_NahradaFP12GM_Component |
| 0x002c4a50 | 52 | 13 | GM_Nahrada | {ctor} | __ct__10GM_NahradaFP15GM_DesignStructP12GM_Component |
| 0x002c4a90 | 96 | 13 | GM_Nahrada | {dtor} | __dt__10GM_NahradaFv |
| 0x002c4af0 | 12 | 13 | GM_Nahrada | GetType | GetType__10GM_NahradaFv |
| 0x002c4b00 | 8 | 13 | GM_Nahrada | CanBeParent | CanBeParent__10GM_NahradaFv |
| 0x002c4b10 | 64 | 13 | GM_Nahrada | GetClientOrigin | GetClientOrigin__10GM_NahradaFRfRf |
| 0x002c4b50 | 20 | 13 | GM_Component | GetClientOrigin | GetClientOrigin__12GM_ComponentFRfRf |
| 0x002c4b70 | 112 | 13 | GM_Nahrada | Draw | Draw__10GM_NahradaFUi |
| 0x002c4be0 | 8 | 13 | GM_Component | GetFlags | GetFlags__12GM_ComponentFv |
| 0x002c4bf0 | 116 | 13 | GM_Component | AfterMove | AfterMove__12GM_ComponentFUi |
| 0x002c4c70 | 8 | 13 | GM_Component | Tick | Tick__12GM_ComponentFUi |
| 0x002c4c80 | 100 | 13 | GM_Component | IsOverClientArea | IsOverClientArea__12GM_ComponentFff |
| 0x002c4cf0 | 8 | 13 | GM_Component | Key | Key__12GM_ComponentFUiUi |
| 0x002c4d00 | 8 | 13 | GM_Component | ScrollToShow | ScrollToShow__12GM_ComponentFP12GM_Component |
| 0x002c4d10 | 8 | 13 | GM_Component | ForceUpdate | ForceUpdate__12GM_ComponentFv |
| 0x002c4d20 | 8 | 13 | GM_Component | GetTextID | GetTextID__12GM_ComponentFv |
| 0x002c4d30 | 8 | 13 | GM_Component | SetText | SetText__12GM_ComponentFPUc |
| 0x002c4d40 | 8 | 13 | GM_Component | SetString | SetString__12GM_ComponentFPUc |
| 0x002c4d50 | 8 | 13 | GM_Component | GetText | GetText__12GM_ComponentFv |
| 0x002c4d60 | 8 | 13 | GM_Component | SetFloat | SetFloat__12GM_ComponentFf |
| 0x002c4d70 | 12 | 13 | GM_Component | GetFloat | GetFloat__12GM_ComponentFv |
| 0x002c4d80 | 8 | 13 | GM_Component | SetDword | SetDword__12GM_ComponentFUi |
| 0x002c4d90 | 8 | 13 | GM_Component | GetDword | GetDword__12GM_ComponentFv |
| 0x002c4da0 | 8 | 13 | GM_Component | SetBool | SetBool__12GM_ComponentFb |
| 0x002c4db0 | 8 | 13 | GM_Component | GetBool | GetBool__12GM_ComponentFv |
| 0x002c4dc0 | 8 | 13 | GM_Component | SetPointer | SetPointer__12GM_ComponentFPv |
| 0x002c4dd0 | 8 | 13 | GM_Component | GetPointer | GetPointer__12GM_ComponentFv |
| 0x002c4de0 | 8 | 13 | GM_Component | SetRangeFloat | SetRangeFloat__12GM_ComponentFff |
| 0x002c4df0 | 8 | 13 | GM_Component | GetRangeFloat | GetRangeFloat__12GM_ComponentFRfRf |
| 0x002c4e00 | 8 | 13 | GM_Component | SetRangeDword | SetRangeDword__12GM_ComponentFUiUi |
| 0x002c4e10 | 8 | 13 | GM_Component | GetRangeDword | GetRangeDword__12GM_ComponentFRUiRUi |
| 0x002c4e20 | 84 | 13 | G_Factory<14GM_PSXTextEdit> | {ctor} | __ct__27G_Factory<14GM_PSXTextEdit>Fv |
| 0x002c4e80 | 16 | 13 | G_GlobalFactory | Register | Register__15G_GlobalFactoryFP13G_FactoryItem |
| 0x002c4e90 | 12 | 13 | GM_PSXTextEdit | GetType | GetType__14GM_PSXTextEditFv |
| 0x002c4ea0 | 84 | 13 | G_Factory<10GM_EditNum> | {ctor} | __ct__23G_Factory<10GM_EditNum>Fv |
| 0x002c4f00 | 12 | 13 | GM_EditNum | GetType | GetType__10GM_EditNumFv |
| 0x002c4f10 | 84 | 13 | G_Factory<19GM_ButtonSelectBool> | {ctor} | __ct__32G_Factory<19GM_ButtonSelectBool>Fv |
| 0x002c4f70 | 12 | 13 | GM_ButtonSelectBool | GetType | GetType__19GM_ButtonSelectBoolFv |
| 0x002c4f80 | 84 | 13 | G_Factory<15GM_ButtonSelect> | {ctor} | __ct__28G_Factory<15GM_ButtonSelect>Fv |
| 0x002c4fe0 | 12 | 13 | GM_ButtonSelect | GetType | GetType__15GM_ButtonSelectFv |
| 0x002c4ff0 | 84 | 13 | G_Factory<11GM_CheckBox> | {ctor} | __ct__24G_Factory<11GM_CheckBox>Fv |
| 0x002c5050 | 12 | 13 | GM_CheckBox | GetType | GetType__11GM_CheckBoxFv |
| 0x002c5060 | 84 | 13 | G_Factory<11GM_TextEdit> | {ctor} | __ct__24G_Factory<11GM_TextEdit>Fv |
| 0x002c50c0 | 12 | 13 | GM_TextEdit | GetType | GetType__11GM_TextEditFv |
| 0x002c50d0 | 84 | 13 | G_Factory<12GM_NetPlayer> | {ctor} | __ct__25G_Factory<12GM_NetPlayer>Fv |
| 0x002c5130 | 12 | 13 | GM_NetPlayer | GetType | GetType__12GM_NetPlayerFv |
| 0x002c5140 | 84 | 13 | G_Factory<16GM_SliderInteger> | {ctor} | __ct__29G_Factory<16GM_SliderInteger>Fv |
| 0x002c51a0 | 12 | 13 | GM_SliderInteger | GetType | GetType__16GM_SliderIntegerFv |
| 0x002c51b0 | 84 | 13 | G_Factory<12GM_SliderNum> | {ctor} | __ct__25G_Factory<12GM_SliderNum>Fv |
| 0x002c5210 | 12 | 13 | GM_SliderNum | GetType | GetType__12GM_SliderNumFv |
| 0x002c5220 | 84 | 13 | G_Factory<9GM_Slider> | {ctor} | __ct__21G_Factory<9GM_Slider>Fv |
| 0x002c5280 | 12 | 13 | GM_Slider | GetType | GetType__9GM_SliderFv |
| 0x002c5290 | 84 | 13 | G_Factory<14GM_ButtonClean> | {ctor} | __ct__27G_Factory<14GM_ButtonClean>Fv |
| 0x002c52f0 | 12 | 13 | GM_ButtonClean | GetType | GetType__14GM_ButtonCleanFv |
| 0x002c5300 | 84 | 13 | G_Factory<15GM_ButtonShrink> | {ctor} | __ct__28G_Factory<15GM_ButtonShrink>Fv |
| 0x002c5360 | 12 | 13 | GM_ButtonShrink | GetType | GetType__15GM_ButtonShrinkFv |
| 0x002c5370 | 84 | 13 | G_Factory<9GM_Button> | {ctor} | __ct__21G_Factory<9GM_Button>Fv |
| 0x002c53d0 | 12 | 13 | GM_Button | GetType | GetType__9GM_ButtonFv |
| 0x002c53e0 | 84 | 13 | G_Factory<16GM_RaceRoundItem> | {ctor} | __ct__29G_Factory<16GM_RaceRoundItem>Fv |
| 0x002c5440 | 12 | 13 | GM_RaceRoundItem | GetType | GetType__16GM_RaceRoundItemFv |
| 0x002c5450 | 84 | 13 | G_Factory<14GM_MenuItemNum> | {ctor} | __ct__27G_Factory<14GM_MenuItemNum>Fv |
| 0x002c54b0 | 12 | 13 | GM_MenuItemNum | GetType | GetType__14GM_MenuItemNumFv |
| 0x002c54c0 | 84 | 13 | G_Factory<14GM_MenuItemMem> | {ctor} | __ct__27G_Factory<14GM_MenuItemMem>Fv |
| 0x002c5520 | 12 | 13 | GM_MenuItemMem | GetType | GetType__14GM_MenuItemMemFv |
| 0x002c5530 | 84 | 13 | G_Factory<11GM_MenuItem> | {ctor} | __ct__24G_Factory<11GM_MenuItem>Fv |
| 0x002c5590 | 12 | 13 | GM_MenuItem | GetType | GetType__11GM_MenuItemFv |
| 0x002c55a0 | 84 | 13 | G_Factory<14GM_ProgressBar> | {ctor} | __ct__27G_Factory<14GM_ProgressBar>Fv |
| 0x002c5600 | 12 | 13 | GM_ProgressBar | GetType | GetType__14GM_ProgressBarFv |
| 0x002c5610 | 84 | 13 | G_Factory<19GM_RectangleTexture> | {ctor} | __ct__32G_Factory<19GM_RectangleTexture>Fv |
| 0x002c5670 | 12 | 13 | GM_RectangleTexture | GetType | GetType__19GM_RectangleTextureFv |
| 0x002c5680 | 84 | 13 | G_Factory<11GM_Orezavac> | {ctor} | __ct__24G_Factory<11GM_Orezavac>Fv |
| 0x002c56e0 | 12 | 13 | GM_Orezavac | GetType | GetType__11GM_OrezavacFv |
| 0x002c56f0 | 84 | 13 | G_Factory<11GM_Triangle> | {ctor} | __ct__24G_Factory<11GM_Triangle>Fv |
| 0x002c5750 | 12 | 13 | GM_Triangle | GetType | GetType__11GM_TriangleFv |
| 0x002c5760 | 84 | 13 | G_Factory<13GM_RectangleB> | {ctor} | __ct__26G_Factory<13GM_RectangleB>Fv |
| 0x002c57c0 | 12 | 13 | GM_RectangleB | GetType | GetType__13GM_RectangleBFv |
| 0x002c57d0 | 84 | 13 | G_Factory<12GM_Rectangle> | {ctor} | __ct__25G_Factory<12GM_Rectangle>Fv |
| 0x002c5830 | 12 | 13 | GM_Rectangle | GetType | GetType__12GM_RectangleFv |
| 0x002c5840 | 84 | 13 | G_Factory<7GM_Line> | {ctor} | __ct__19G_Factory<7GM_Line>Fv |
| 0x002c58a0 | 12 | 13 | GM_Line | GetType | GetType__7GM_LineFv |
| 0x002c58b0 | 84 | 13 | G_Factory<17GM_StaticTextTime> | {ctor} | __ct__30G_Factory<17GM_StaticTextTime>Fv |
| 0x002c5910 | 12 | 13 | GM_StaticTextTime | GetType | GetType__17GM_StaticTextTimeFv |
| 0x002c5920 | 84 | 13 | G_Factory<16GM_StaticTextNum> | {ctor} | __ct__29G_Factory<16GM_StaticTextNum>Fv |
| 0x002c5980 | 12 | 13 | GM_StaticTextNum | GetType | GetType__16GM_StaticTextNumFv |
| 0x002c5990 | 84 | 13 | G_Factory<17GM_StaticTextMem2> | {ctor} | __ct__30G_Factory<17GM_StaticTextMem2>Fv |
| 0x002c59f0 | 12 | 13 | GM_StaticTextMem2 | GetType | GetType__17GM_StaticTextMem2Fv |
| 0x002c5a00 | 84 | 13 | G_Factory<16GM_StaticTextMem> | {ctor} | __ct__29G_Factory<16GM_StaticTextMem>Fv |
| 0x002c5a60 | 12 | 13 | GM_StaticTextMem | GetType | GetType__16GM_StaticTextMemFv |
| 0x002c5a70 | 84 | 13 | G_Factory<16GM_StaticTextFit> | {ctor} | __ct__29G_Factory<16GM_StaticTextFit>Fv |
| 0x002c5ad0 | 12 | 13 | GM_StaticTextFit | GetType | GetType__16GM_StaticTextFitFv |
| 0x002c5ae0 | 84 | 13 | G_Factory<13GM_StaticText> | {ctor} | __ct__26G_Factory<13GM_StaticText>Fv |
| 0x002c5b40 | 12 | 13 | GM_StaticText | GetType | GetType__13GM_StaticTextFv |
| 0x002c5b50 | 84 | 13 | G_Factory<7GM_Mapa> | {ctor} | __ct__19G_Factory<7GM_Mapa>Fv |
| 0x002c5bb0 | 12 | 13 | GM_Mapa | GetType | GetType__7GM_MapaFv |
| 0x002c5bc0 | 84 | 13 | G_Factory<11GM_PS2Help2> | {ctor} | __ct__24G_Factory<11GM_PS2Help2>Fv |
| 0x002c5c20 | 12 | 13 | GM_PS2Help2 | GetType | GetType__11GM_PS2Help2Fv |
| 0x002c5c30 | 84 | 13 | G_Factory<10GM_PS2Help> | {ctor} | __ct__23G_Factory<10GM_PS2Help>Fv |
| 0x002c5c90 | 12 | 13 | GM_PS2Help | GetType | GetType__10GM_PS2HelpFv |
| 0x002c5ca0 | 84 | 13 | G_Factory<15GM_WindowScroll> | {ctor} | __ct__28G_Factory<15GM_WindowScroll>Fv |
| 0x002c5d00 | 12 | 13 | GM_WindowScroll | GetType | GetType__15GM_WindowScrollFv |
| 0x002c5d10 | 84 | 13 | G_Factory<17GM_WindowTutorial> | {ctor} | __ct__30G_Factory<17GM_WindowTutorial>Fv |
| 0x002c5d70 | 12 | 13 | GM_WindowTutorial | GetType | GetType__17GM_WindowTutorialFv |
| 0x002c5d80 | 84 | 13 | G_Factory<14GM_WindowClean> | {ctor} | __ct__27G_Factory<14GM_WindowClean>Fv |
| 0x002c5de0 | 12 | 13 | GM_WindowClean | GetType | GetType__14GM_WindowCleanFv |
| 0x002c5df0 | 84 | 13 | G_Factory<9GM_Window> | {ctor} | __ct__21G_Factory<9GM_Window>Fv |
| 0x002c5e50 | 12 | 13 | GM_Window | GetType | GetType__9GM_WindowFv |
| 0x002c5e60 | 84 | 13 | G_Factory<7GM_UpUp> | {ctor} | __ct__19G_Factory<7GM_UpUp>Fv |
| 0x002c5ec0 | 12 | 13 | GM_UpUp | GetType | GetType__7GM_UpUpFv |
| 0x002c5ed0 | 84 | 13 | G_Factory<10GM_Nahrada> | {ctor} | __ct__23G_Factory<10GM_Nahrada>Fv |
| 0x002c5f30 | 84 | 13 | G_Factory<12GM_Component> | {ctor} | __ct__25G_Factory<12GM_Component>Fv |
| 0x002c5f90 | 12 | 13 | GM_Component | GetType | GetType__12GM_ComponentFv |
| 0x002c5fa0 | 56 | 13 | S_quat | {ctor} | __ct__6S_quatFffff |
| 0x002c5fe0 | 96 | 13 | GM_UpUp | {dtor} | __dt__7GM_UpUpFv |
| 0x002c6040 | 88 | 13 | GM_UpUp | Draw | Draw__7GM_UpUpFUi |
| 0x002c60a0 | 8 | 13 | GM_Component | CanBeParent | CanBeParent__12GM_ComponentFv |
| 0x002c60b0 | 96 | 13 | GM_Window | {dtor} | __dt__9GM_WindowFv |
| 0x002c6110 | 8 | 13 | GM_Window | CanBeParent | CanBeParent__9GM_WindowFv |
| 0x002c6120 | 44 | 13 | GM_Window | GetClientOrigin | GetClientOrigin__9GM_WindowFRfRf |
| 0x002c6150 | 860 | 13 | GM_Window | Draw | Draw__9GM_WindowFUi |
| 0x002c64b0 | 96 | 13 | GM_WindowClean | {dtor} | __dt__14GM_WindowCleanFv |
| 0x002c6510 | 8 | 13 | GM_WindowClean | CanBeParent | CanBeParent__14GM_WindowCleanFv |
| 0x002c6520 | 8 | 13 | GM_WindowClean | Draw | Draw__14GM_WindowCleanFUi |
| 0x002c6530 | 96 | 13 | GM_WindowTutorial | {dtor} | __dt__17GM_WindowTutorialFv |
| 0x002c6590 | 8 | 13 | GM_WindowTutorial | CanBeParent | CanBeParent__17GM_WindowTutorialFv |
| 0x002c65a0 | 180 | 13 | GM_WindowTutorial | Draw | Draw__17GM_WindowTutorialFUi |
| 0x002c6660 | 96 | 13 | GM_WindowScroll | {dtor} | __dt__15GM_WindowScrollFv |
| 0x002c66c0 | 8 | 13 | GM_WindowScroll | CanBeParent | CanBeParent__15GM_WindowScrollFv |
| 0x002c66d0 | 28 | 13 | GM_WindowScroll | GetClientOrigin | GetClientOrigin__15GM_WindowScrollFRfRf |
| 0x002c66f0 | 112 | 13 | GM_WindowScroll | IsOverActive | IsOverActive__15GM_WindowScrollFff |
| 0x002c6760 | 64 | 13 | GM_WindowScroll | SetFloat | SetFloat__15GM_WindowScrollFf |
| 0x002c67a0 | 100 | 13 | GM_WindowScroll | SetRangeFloat | SetRangeFloat__15GM_WindowScrollFff |
| 0x002c6810 | 8 | 13 | GM_WindowScroll | GetFloat | GetFloat__15GM_WindowScrollFv |
| 0x002c6820 | 16 | 13 | GM_WindowScroll | GetRangeFloat | GetRangeFloat__15GM_WindowScrollFRfRf |
| 0x002c6830 | 556 | 13 | GM_WindowScroll | Draw | Draw__15GM_WindowScrollFUi |
| 0x002c6a60 | 424 | 13 | GM_WindowScroll | Tick | Tick__15GM_WindowScrollFUi |
| 0x002c6c10 | 8 | 13 | gif::vector<P12GM_Component,8> | end | end__Q23gif25vector<P12GM_Component,8>Fv |
| 0x002c6c20 | 8 | 13 | gif::vector<P12GM_Component,8> | begin | begin__Q23gif25vector<P12GM_Component,8>Fv |
| 0x002c6c30 | 132 | 13 | GM_WindowScroll | Key | Key__15GM_WindowScrollFUiUi |
| 0x002c6cc0 | 96 | 13 | GM_PS2Help | {dtor} | __dt__10GM_PS2HelpFv |
| 0x002c6d20 | 264 | 13 | GM_PS2Help | Draw | Draw__10GM_PS2HelpFUi |
| 0x002c6e30 | 96 | 13 | GM_PS2Help2 | {dtor} | __dt__11GM_PS2Help2Fv |
| 0x002c6e90 | 264 | 13 | GM_PS2Help2 | Draw | Draw__11GM_PS2Help2FUi |
| 0x002c6fa0 | 96 | 13 | GM_Mapa | {dtor} | __dt__7GM_MapaFv |
| 0x002c7000 | 28 | 13 | GM_Mapa | Draw | Draw__7GM_MapaFUi |
| 0x002c7020 | 96 | 13 | GM_StaticText | {dtor} | __dt__13GM_StaticTextFv |
| 0x002c7080 | 244 | 13 | GM_StaticText | Draw | Draw__13GM_StaticTextFUi |
| 0x002c7180 | 96 | 13 | GM_StaticTextFit | {dtor} | __dt__16GM_StaticTextFitFv |
| 0x002c71e0 | 244 | 13 | GM_StaticTextFit | Draw | Draw__16GM_StaticTextFitFUi |
| 0x002c72e0 | 96 | 13 | GM_StaticTextMem | {dtor} | __dt__16GM_StaticTextMemFv |
| 0x002c7340 | 52 | 13 | GM_StaticTextMem | SetText | SetText__16GM_StaticTextMemFPUc |
| 0x002c7380 | 244 | 13 | GM_StaticTextMem | Draw | Draw__16GM_StaticTextMemFUi |
| 0x002c7480 | 96 | 13 | GM_StaticTextMem2 | {dtor} | __dt__17GM_StaticTextMem2Fv |
| 0x002c74e0 | 52 | 13 | GM_StaticTextMem2 | SetText | SetText__17GM_StaticTextMem2FPUc |
| 0x002c7520 | 244 | 13 | GM_StaticTextMem2 | Draw | Draw__17GM_StaticTextMem2FUi |
| 0x002c7620 | 96 | 13 | GM_StaticTextNum | {dtor} | __dt__16GM_StaticTextNumFv |
| 0x002c7680 | 68 | 13 | GM_StaticTextNum | SetDword | SetDword__16GM_StaticTextNumFUi |
| 0x002c76d0 | 256 | 13 | GM_StaticTextNum | Draw | Draw__16GM_StaticTextNumFUi |
| 0x002c77d0 | 96 | 13 | GM_StaticTextTime | {dtor} | __dt__17GM_StaticTextTimeFv |
| 0x002c7830 | 240 | 13 | GM_StaticTextTime | SetDword | SetDword__17GM_StaticTextTimeFUi |
| 0x002c7920 | 256 | 13 | GM_StaticTextTime | Draw | Draw__17GM_StaticTextTimeFUi |
| 0x002c7a20 | 96 | 13 | GM_Line | {dtor} | __dt__7GM_LineFv |
| 0x002c7a80 | 92 | 13 | GM_Line | Draw | Draw__7GM_LineFUi |
| 0x002c7ae0 | 96 | 13 | GM_Rectangle | {dtor} | __dt__12GM_RectangleFv |
| 0x002c7b40 | 188 | 13 | GM_Rectangle | Draw | Draw__12GM_RectangleFUi |
| 0x002c7c00 | 96 | 13 | GM_RectangleB | {dtor} | __dt__13GM_RectangleBFv |
| 0x002c7c60 | 188 | 13 | GM_RectangleB | Draw | Draw__13GM_RectangleBFUi |
| 0x002c7d20 | 96 | 13 | GM_Triangle | {dtor} | __dt__11GM_TriangleFv |
| 0x002c7d80 | 192 | 13 | GM_Triangle | Draw | Draw__11GM_TriangleFUi |
| 0x002c7e40 | 96 | 13 | GM_Orezavac | {dtor} | __dt__11GM_OrezavacFv |
| 0x002c7ea0 | 60 | 13 | GM_Orezavac | Draw | Draw__11GM_OrezavacFUi |
| 0x002c7ee0 | 120 | 13 | GM_RectangleTexture | {dtor} | __dt__19GM_RectangleTextureFv |
| 0x002c7f60 | 100 | 13 | GM_RectangleTexture | SetString | SetString__19GM_RectangleTextureFPUc |
| 0x002c7fd0 | 8 | 13 | GM_RectangleTexture | SetPointer | SetPointer__19GM_RectangleTextureFPv |
| 0x002c7fe0 | 8 | 13 | GM_RectangleTexture | GetPointer | GetPointer__19GM_RectangleTextureFv |
| 0x002c7ff0 | 56 | 13 | GM_RectangleTexture | Draw | Draw__19GM_RectangleTextureFUi |
| 0x002c8030 | 96 | 13 | GM_ProgressBar | {dtor} | __dt__14GM_ProgressBarFv |
| 0x002c8090 | 8 | 13 | GM_ProgressBar | SetFloat | SetFloat__14GM_ProgressBarFf |
| 0x002c80a0 | 104 | 13 | GM_ProgressBar | Draw | Draw__14GM_ProgressBarFUi |
| 0x002c8110 | 96 | 13 | GM_MenuItem | {dtor} | __dt__11GM_MenuItemFv |
| 0x002c8170 | 248 | 13 | GM_MenuItem | Draw | Draw__11GM_MenuItemFUi |
| 0x002c8270 | 48 | 13 | GM_MenuItem | Key | Key__11GM_MenuItemFUiUi |
| 0x002c82a0 | 96 | 13 | GM_MenuItemMem | {dtor} | __dt__14GM_MenuItemMemFv |
| 0x002c8300 | 52 | 13 | GM_MenuItemMem | SetText | SetText__14GM_MenuItemMemFPUc |
| 0x002c8340 | 248 | 13 | GM_MenuItemMem | Draw | Draw__14GM_MenuItemMemFUi |
| 0x002c8440 | 48 | 13 | GM_MenuItemMem | Key | Key__14GM_MenuItemMemFUiUi |
| 0x002c8470 | 96 | 13 | GM_MenuItemNum | {dtor} | __dt__14GM_MenuItemNumFv |
| 0x002c84d0 | 72 | 13 | GM_MenuItemNum | SetRangeDword | SetRangeDword__14GM_MenuItemNumFUiUi |
| 0x002c8520 | 68 | 13 | GM_MenuItemNum | SetDword | SetDword__14GM_MenuItemNumFUi |
| 0x002c8570 | 8 | 13 | GM_MenuItemNum | GetDword | GetDword__14GM_MenuItemNumFv |
| 0x002c8580 | 600 | 13 | GM_MenuItemNum | Draw | Draw__14GM_MenuItemNumFUi |
| 0x002c87e0 | 292 | 13 | GM_MenuItemNum | Key | Key__14GM_MenuItemNumFUiUi |
| 0x002c8910 | 96 | 13 | GM_RaceRoundItem | {dtor} | __dt__16GM_RaceRoundItemFv |
| 0x002c8970 | 324 | 13 | GM_RaceRoundItem | SetBool | SetBool__16GM_RaceRoundItemFb |
| 0x002c8ac0 | 8 | 13 | GM_RaceRoundItem | SetDword | SetDword__16GM_RaceRoundItemFUi |
| 0x002c8ad0 | 408 | 13 | GM_RaceRoundItem | Draw | Draw__16GM_RaceRoundItemFUi |
| 0x002c8c70 | 48 | 13 | GM_RaceRoundItem | Key | Key__16GM_RaceRoundItemFUiUi |
| 0x002c8ca0 | 96 | 13 | GM_Button | {dtor} | __dt__9GM_ButtonFv |
| 0x002c8d00 | 576 | 13 | GM_Button | Draw | Draw__9GM_ButtonFUi |
| 0x002c8f40 | 48 | 13 | GM_Button | Key | Key__9GM_ButtonFUiUi |
| 0x002c8f70 | 96 | 13 | GM_ButtonShrink | {dtor} | __dt__15GM_ButtonShrinkFv |
| 0x002c8fd0 | 576 | 13 | GM_ButtonShrink | Draw | Draw__15GM_ButtonShrinkFUi |
| 0x002c9210 | 48 | 13 | GM_ButtonShrink | Key | Key__15GM_ButtonShrinkFUiUi |
| 0x002c9240 | 96 | 13 | GM_ButtonClean | {dtor} | __dt__14GM_ButtonCleanFv |
| 0x002c92a0 | 8 | 13 | GM_ButtonClean | CanBeParent | CanBeParent__14GM_ButtonCleanFv |
| 0x002c92b0 | 8 | 13 | GM_ButtonClean | Draw | Draw__14GM_ButtonCleanFUi |
| 0x002c92c0 | 96 | 13 | GM_Slider | {dtor} | __dt__9GM_SliderFv |
| 0x002c9320 | 8 | 13 | GM_Slider | ForceUpdate | ForceUpdate__9GM_SliderFv |
| 0x002c9330 | 20 | 13 | GM_Slider | SetFloat | SetFloat__9GM_SliderFf |
| 0x002c9350 | 8 | 13 | GM_Slider | GetFloat | GetFloat__9GM_SliderFv |
| 0x002c9360 | 692 | 13 | GM_Slider | Draw | Draw__9GM_SliderFUi |
| 0x002c9620 | 220 | 13 | GM_Slider | Key | Key__9GM_SliderFUiUi |
| 0x002c9700 | 96 | 13 | GM_SliderNum | {dtor} | __dt__12GM_SliderNumFv |
| 0x002c9760 | 8 | 13 | GM_SliderNum | ForceUpdate | ForceUpdate__12GM_SliderNumFv |
| 0x002c9770 | 20 | 13 | GM_SliderNum | SetFloat | SetFloat__12GM_SliderNumFf |
| 0x002c9790 | 8 | 13 | GM_SliderNum | GetFloat | GetFloat__12GM_SliderNumFv |
| 0x002c97a0 | 908 | 13 | GM_SliderNum | Draw | Draw__12GM_SliderNumFUi |
| 0x002c9b30 | 360 | 13 | GM_SliderNum | Key | Key__12GM_SliderNumFUiUi |
| 0x002c9ca0 | 96 | 13 | GM_SliderInteger | {dtor} | __dt__16GM_SliderIntegerFv |
| 0x002c9d00 | 8 | 13 | GM_SliderInteger | ForceUpdate | ForceUpdate__16GM_SliderIntegerFv |
| 0x002c9d10 | 12 | 13 | GM_SliderInteger | SetRangeDword | SetRangeDword__16GM_SliderIntegerFUiUi |
| 0x002c9d20 | 20 | 13 | GM_SliderInteger | SetDword | SetDword__16GM_SliderIntegerFUi |
| 0x002c9d40 | 8 | 13 | GM_SliderInteger | GetDword | GetDword__16GM_SliderIntegerFv |
| 0x002c9d50 | 704 | 13 | GM_SliderInteger | Draw | Draw__16GM_SliderIntegerFUi |
| 0x002ca010 | 188 | 13 | GM_SliderInteger | Key | Key__16GM_SliderIntegerFUiUi |
| 0x002ca0d0 | 96 | 13 | GM_NetPlayer | {dtor} | __dt__12GM_NetPlayerFv |
| 0x002ca130 | 8 | 13 | GM_NetPlayer | SetPointer | SetPointer__12GM_NetPlayerFPv |
| 0x002ca140 | 8 | 13 | GM_NetPlayer | GetPointer | GetPointer__12GM_NetPlayerFv |
| 0x002ca150 | 856 | 13 | GM_NetPlayer | Draw | Draw__12GM_NetPlayerFUi |
| 0x002ca4b0 | 8 | 13 | GM_NetPlayer | Key | Key__12GM_NetPlayerFUiUi |
| 0x002ca4c0 | 96 | 13 | GM_CheckBox | {dtor} | __dt__11GM_CheckBoxFv |
| 0x002ca520 | 648 | 13 | GM_CheckBox | Draw | Draw__11GM_CheckBoxFUi |
| 0x002ca7b0 | 8 | 13 | GM_CheckBox | SetBool | SetBool__11GM_CheckBoxFb |
| 0x002ca7c0 | 8 | 13 | GM_CheckBox | GetBool | GetBool__11GM_CheckBoxFv |
| 0x002ca7d0 | 56 | 13 | GM_CheckBox | Key | Key__11GM_CheckBoxFUiUi |
| 0x002ca810 | 96 | 13 | GM_ButtonSelect | {dtor} | __dt__15GM_ButtonSelectFv |
| 0x002ca870 | 12 | 13 | GM_ButtonSelect | SetRangeDword | SetRangeDword__15GM_ButtonSelectFUiUi |
| 0x002ca880 | 188 | 13 | GM_ButtonSelect | Draw | Draw__15GM_ButtonSelectFUi |
| 0x002ca940 | 220 | 13 | GM_ButtonSelect | Key | Key__15GM_ButtonSelectFUiUi |
| 0x002caa20 | 96 | 13 | GM_ButtonSelectBool | {dtor} | __dt__19GM_ButtonSelectBoolFv |
| 0x002caa80 | 80 | 13 | GM_ButtonSelectBool | SetRangeDword | SetRangeDword__19GM_ButtonSelectBoolFUiUi |
| 0x002caad0 | 76 | 13 | GM_ButtonSelectBool | SetBool | SetBool__19GM_ButtonSelectBoolFb |
| 0x002cab20 | 8 | 13 | GM_ButtonSelectBool | GetBool | GetBool__19GM_ButtonSelectBoolFv |
| 0x002cab30 | 188 | 13 | GM_ButtonSelectBool | Draw | Draw__19GM_ButtonSelectBoolFUi |
| 0x002cabf0 | 172 | 13 | GM_ButtonSelectBool | Key | Key__19GM_ButtonSelectBoolFUiUi |
| 0x002caca0 | 96 | 13 | GM_EditNum | {dtor} | __dt__10GM_EditNumFv |
| 0x002cad00 | 76 | 13 | GM_EditNum | SetRangeDword | SetRangeDword__10GM_EditNumFUiUi |
| 0x002cad50 | 72 | 13 | GM_EditNum | SetDword | SetDword__10GM_EditNumFUi |
| 0x002cada0 | 8 | 13 | GM_EditNum | GetDword | GetDword__10GM_EditNumFv |
| 0x002cadb0 | 336 | 13 | GM_EditNum | Draw | Draw__10GM_EditNumFUi |
| 0x002caf00 | 260 | 13 | GM_EditNum | Key | Key__10GM_EditNumFUiUi |
| 0x002cb010 | 68 | 13 | GM_TextEdit | SetText | SetText__11GM_TextEditFPUc |
| 0x002cb060 | 8 | 13 | GM_TextEdit | GetText | GetText__11GM_TextEditFv |
| 0x002cb070 | 44 | 13 | GM_TextEdit | SetDword | SetDword__11GM_TextEditFUi |
| 0x002cb0a0 | 180 | 13 | GM_TextEdit | Key | Key__11GM_TextEditFUiUi |
| 0x002cb160 | 176 | 13 | GM_TextEdit | Draw | Draw__11GM_TextEditFUi |
| 0x002cb210 | 96 | 13 | GM_PSXTextEdit | {dtor} | __dt__14GM_PSXTextEditFv |
| 0x002cb270 | 368 | 13 | GM_PSXTextEdit | Key | Key__14GM_PSXTextEditFUiUi |
| 0x002cb3e0 | 72 | 13 | G_Factory<11GM_ComboBox> | Make | Make__24G_Factory<11GM_ComboBox>FP15GM_DesignStructP12GM_Component |
| 0x002cb430 | 56 | 13 | G_Factory<11GM_ComboBox> | MakeDup | MakeDup__24G_Factory<11GM_ComboBox>FP12GM_Component |
| 0x002cb470 | 72 | 13 | G_Factory<14GM_ComboWindow> | Make | Make__27G_Factory<14GM_ComboWindow>FP15GM_DesignStructP12GM_Component |
| 0x002cb4c0 | 56 | 13 | G_Factory<14GM_ComboWindow> | MakeDup | MakeDup__27G_Factory<14GM_ComboWindow>FP12GM_Component |
| 0x002cb500 | 72 | 13 | G_Factory<10GM_ListBox> | Make | Make__23G_Factory<10GM_ListBox>FP15GM_DesignStructP12GM_Component |
| 0x002cb550 | 56 | 13 | G_Factory<10GM_ListBox> | MakeDup | MakeDup__23G_Factory<10GM_ListBox>FP12GM_Component |
| 0x002cb590 | 72 | 13 | G_Factory<11GM_ListCars> | Make | Make__24G_Factory<11GM_ListCars>FP15GM_DesignStructP12GM_Component |
| 0x002cb5e0 | 56 | 13 | G_Factory<11GM_ListCars> | MakeDup | MakeDup__24G_Factory<11GM_ListCars>FP12GM_Component |
| 0x002cb620 | 72 | 13 | G_Factory<13GM_ViewScroll> | Make | Make__26G_Factory<13GM_ViewScroll>FP15GM_DesignStructP12GM_Component |
| 0x002cb670 | 56 | 13 | G_Factory<13GM_ViewScroll> | MakeDup | MakeDup__26G_Factory<13GM_ViewScroll>FP12GM_Component |
| 0x002cb6b0 | 72 | 13 | G_Factory<12GM_TextInput> | Make | Make__25G_Factory<12GM_TextInput>FP15GM_DesignStructP12GM_Component |
| 0x002cb700 | 56 | 13 | G_Factory<12GM_TextInput> | MakeDup | MakeDup__25G_Factory<12GM_TextInput>FP12GM_Component |
| 0x002cb740 | 72 | 13 | G_Factory<16GM_StaticTextBig> | Make | Make__29G_Factory<16GM_StaticTextBig>FP15GM_DesignStructP12GM_Component |
| 0x002cb790 | 56 | 13 | G_Factory<16GM_StaticTextBig> | MakeDup | MakeDup__29G_Factory<16GM_StaticTextBig>FP12GM_Component |
| 0x002cb7d0 | 72 | 13 | G_Factory<22GM_StaticTextBig2Color> | Make | Make__35G_Factory<22GM_StaticTextBig2Color>FP15GM_DesignStructP12GM_Component |
| 0x002cb820 | 56 | 13 | G_Factory<22GM_StaticTextBig2Color> | MakeDup | MakeDup__35G_Factory<22GM_StaticTextBig2Color>FP12GM_Component |
| 0x002cb860 | 72 | 13 | G_Factory<10GM_Karusel> | Make | Make__23G_Factory<10GM_Karusel>FP15GM_DesignStructP12GM_Component |
| 0x002cb8b0 | 56 | 13 | G_Factory<10GM_Karusel> | MakeDup | MakeDup__23G_Factory<10GM_Karusel>FP12GM_Component |
| 0x002cb8f0 | 72 | 13 | G_Factory<11GM_ColorSel> | Make | Make__24G_Factory<11GM_ColorSel>FP15GM_DesignStructP12GM_Component |
| 0x002cb940 | 56 | 13 | G_Factory<11GM_ColorSel> | MakeDup | MakeDup__24G_Factory<11GM_ColorSel>FP12GM_Component |
| 0x002cb980 | 88 | 13 | GM_ColorSel | {ctor} | __ct__11GM_ColorSelFP12GM_Component |
| 0x002cb9e0 | 100 | 13 | GM_ColorSel | {ctor} | __ct__11GM_ColorSelFP15GM_DesignStructP12GM_Component |
| 0x002cba50 | 88 | 13 | GM_Karusel | {ctor} | __ct__10GM_KaruselFP12GM_Component |
| 0x002cbab0 | 100 | 13 | GM_Karusel | {ctor} | __ct__10GM_KaruselFP15GM_DesignStructP12GM_Component |
| 0x002cbb20 | 72 | 13 | GM_StaticTextBig2Color | {ctor} | __ct__22GM_StaticTextBig2ColorFP12GM_Component |
| 0x002cbb70 | 264 | 13 | GM_StaticTextBig2Color | AddText | AddText__22GM_StaticTextBig2ColorFPUc |
| 0x002cbc80 | 12 | 13 | GM_StaticTextBig2Color | Clear | Clear__22GM_StaticTextBig2ColorFv |
| 0x002cbc90 | 72 | 13 | GM_StaticTextBig2Color | {ctor} | __ct__22GM_StaticTextBig2ColorFP15GM_DesignStructP12GM_Component |
| 0x002cbce0 | 72 | 13 | GM_StaticTextBig | {ctor} | __ct__16GM_StaticTextBigFP12GM_Component |
| 0x002cbd30 | 264 | 13 | GM_StaticTextBig | AddText | AddText__16GM_StaticTextBigFPUc |
| 0x002cbe40 | 12 | 13 | GM_StaticTextBig | Clear | Clear__16GM_StaticTextBigFv |
| 0x002cbe50 | 72 | 13 | GM_StaticTextBig | {ctor} | __ct__16GM_StaticTextBigFP15GM_DesignStructP12GM_Component |
| 0x002cbea0 | 52 | 13 | GM_TextInput | {ctor} | __ct__12GM_TextInputFP12GM_Component |
| 0x002cbee0 | 52 | 13 | GM_TextInput | {ctor} | __ct__12GM_TextInputFP15GM_DesignStructP12GM_Component |
| 0x002cbf20 | 60 | 13 | GM_ViewScroll | {ctor} | __ct__13GM_ViewScrollFP12GM_Component |
| 0x002cbf60 | 60 | 13 | GM_ViewScroll | {ctor} | __ct__13GM_ViewScrollFP15GM_DesignStructP12GM_Component |
| 0x002cbfa0 | 88 | 13 | GM_ListCars | {ctor} | __ct__11GM_ListCarsFP12GM_Component |
| 0x002cc000 | 100 | 13 | GM_ListCars | {ctor} | __ct__11GM_ListCarsFP15GM_DesignStructP12GM_Component |
| 0x002cc070 | 68 | 13 | GM_ListBox | {ctor} | __ct__10GM_ListBoxFP12GM_Component |
| 0x002cc0c0 | 80 | 13 | GM_ListBox | {ctor} | __ct__10GM_ListBoxFP15GM_DesignStructP12GM_Component |
| 0x002cc110 | 52 | 13 | GM_ComboWindow | {ctor} | __ct__14GM_ComboWindowFP12GM_Component |
| 0x002cc150 | 52 | 13 | GM_ComboWindow | {ctor} | __ct__14GM_ComboWindowFP15GM_DesignStructP12GM_Component |
| 0x002cc190 | 60 | 13 | GM_ComboBox | {ctor} | __ct__11GM_ComboBoxFP12GM_Component |
| 0x002cc1d0 | 72 | 13 | GM_ComboBox | {ctor} | __ct__11GM_ComboBoxFP15GM_DesignStructP12GM_Component |
| 0x002cc220 | 96 | 13 | GM_ComboBox | {dtor} | __dt__11GM_ComboBoxFv |
| 0x002cc280 | 12 | 13 | GM_ComboBox | GetType | GetType__11GM_ComboBoxFv |
| 0x002cc290 | 84 | 13 | GM_ComboBox | SetPointer | SetPointer__11GM_ComboBoxFPv |
| 0x002cc2f0 | 8 | 13 | gif::vector<17GM_ComboBoxStruct,8> | begin | begin__Q23gif29vector<17GM_ComboBoxStruct,8>Fv |
| 0x002cc300 | 36 | 13 | gif::vector<17GM_ComboBoxStruct,8> | size | size__Q23gif29vector<17GM_ComboBoxStruct,8>CFv |
| 0x002cc330 | 8 | 13 | GM_ComboBox | GetPointer | GetPointer__11GM_ComboBoxFv |
| 0x002cc340 | 144 | 13 | GM_ComboBox | SetDword | SetDword__11GM_ComboBoxFUi |
| 0x002cc3d0 | 8 | 13 | gif::vector<17GM_ComboBoxStruct,8> | end | end__Q23gif29vector<17GM_ComboBoxStruct,8>Fv |
| 0x002cc3e0 | 32 | 13 | GM_ComboBox | GetDword | GetDword__11GM_ComboBoxFv |
| 0x002cc400 | 32 | 13 | GM_ComboBox | GetText | GetText__11GM_ComboBoxFv |
| 0x002cc420 | 552 | 13 | GM_ComboBox | Draw | Draw__11GM_ComboBoxFUi |
| 0x002cc650 | 112 | 13 | GM_ComboBox | Key | Key__11GM_ComboBoxFUiUi |
| 0x002cc6c0 | 96 | 13 | GM_ComboMenu | {ctor} | __ct__12GM_ComboMenuFP12GM_Component |
| 0x002cc720 | 84 | 13 | G_Factory<11GM_ColorSel> | {ctor} | __ct__24G_Factory<11GM_ColorSel>Fv |
| 0x002cc780 | 12 | 13 | GM_ColorSel | GetType | GetType__11GM_ColorSelFv |
| 0x002cc790 | 84 | 13 | G_Factory<10GM_Karusel> | {ctor} | __ct__23G_Factory<10GM_Karusel>Fv |
| 0x002cc7f0 | 12 | 13 | GM_Karusel | GetType | GetType__10GM_KaruselFv |
| 0x002cc800 | 84 | 13 | G_Factory<22GM_StaticTextBig2Color> | {ctor} | __ct__35G_Factory<22GM_StaticTextBig2Color>Fv |
| 0x002cc860 | 12 | 13 | GM_StaticTextBig2Color | GetType | GetType__22GM_StaticTextBig2ColorFv |
| 0x002cc870 | 84 | 13 | G_Factory<16GM_StaticTextBig> | {ctor} | __ct__29G_Factory<16GM_StaticTextBig>Fv |
| 0x002cc8d0 | 12 | 13 | GM_StaticTextBig | GetType | GetType__16GM_StaticTextBigFv |
| 0x002cc8e0 | 84 | 13 | G_Factory<12GM_TextInput> | {ctor} | __ct__25G_Factory<12GM_TextInput>Fv |
| 0x002cc940 | 12 | 13 | GM_TextInput | GetType | GetType__12GM_TextInputFv |
| 0x002cc950 | 84 | 13 | G_Factory<13GM_ViewScroll> | {ctor} | __ct__26G_Factory<13GM_ViewScroll>Fv |
| 0x002cc9b0 | 12 | 13 | GM_ViewScroll | GetType | GetType__13GM_ViewScrollFv |
| 0x002cc9c0 | 84 | 13 | G_Factory<11GM_ListCars> | {ctor} | __ct__24G_Factory<11GM_ListCars>Fv |
| 0x002cca20 | 12 | 13 | GM_ListCars | GetType | GetType__11GM_ListCarsFv |
| 0x002cca30 | 84 | 13 | G_Factory<10GM_ListBox> | {ctor} | __ct__23G_Factory<10GM_ListBox>Fv |
| 0x002cca90 | 12 | 13 | GM_ListBox | GetType | GetType__10GM_ListBoxFv |
| 0x002ccaa0 | 84 | 13 | G_Factory<14GM_ComboWindow> | {ctor} | __ct__27G_Factory<14GM_ComboWindow>Fv |
| 0x002ccb00 | 12 | 13 | GM_ComboWindow | GetType | GetType__14GM_ComboWindowFv |
| 0x002ccb10 | 84 | 13 | G_Factory<11GM_ComboBox> | {ctor} | __ct__24G_Factory<11GM_ComboBox>Fv |
| 0x002ccb70 | 96 | 13 | GM_ComboMenu | {dtor} | __dt__12GM_ComboMenuFv |
| 0x002ccbd0 | 12 | 13 | GM_ComboMenu | OnBeforeCreate | OnBeforeCreate__12GM_ComboMenuFv |
| 0x002ccbe0 | 492 | 13 | GM_ComboMenu | OnCreate | OnCreate__12GM_ComboMenuFv |
| 0x002ccdd0 | 16 | 13 | gif::vector<17GM_ComboBoxStruct,8> | operator[] | __vc__Q23gif29vector<17GM_ComboBoxStruct,8>Fi |
| 0x002ccde0 | 56 | 13 | GM_ComboMenu | OnClose | OnClose__12GM_ComboMenuFv |
| 0x002cce20 | 8 | 13 | GM_Menu | OnClick | OnClick__7GM_MenuFUi |
| 0x002cce30 | 156 | 13 | GM_ComboMenu | OnClick | OnClick__12GM_ComboMenuFUi |
| 0x002cced0 | 44 | 13 | GM_Menu | OnKey | OnKey__7GM_MenuFUcUi |
| 0x002ccf00 | 8 | 13 | GM_Menu | OnMouse | OnMouse__7GM_MenuFiiiUiUi |
| 0x002ccf10 | 8 | 13 | GM_Menu | OnTick | OnTick__7GM_MenuFUi |
| 0x002ccf20 | 96 | 13 | GM_ComboWindow | {dtor} | __dt__14GM_ComboWindowFv |
| 0x002ccf80 | 8 | 13 | GM_ComboWindow | CanBeParent | CanBeParent__14GM_ComboWindowFv |
| 0x002ccf90 | 120 | 13 | GM_ComboWindow | Draw | Draw__14GM_ComboWindowFUi |
| 0x002cd010 | 96 | 13 | GM_ListBox | {dtor} | __dt__10GM_ListBoxFv |
| 0x002cd070 | 152 | 13 | GM_ListBox | SetPointer | SetPointer__10GM_ListBoxFPv |
| 0x002cd110 | 52 | 13 | gif::vector<16GM_ListBoxStruct,8> | size | size__Q23gif28vector<16GM_ListBoxStruct,8>CFv |
| 0x002cd150 | 8 | 13 | GM_ListBox | GetPointer | GetPointer__10GM_ListBoxFv |
| 0x002cd160 | 708 | 13 | GM_ListBox | Draw | Draw__10GM_ListBoxFUi |
| 0x002cd430 | 8 | 13 | gif::vector<16GM_ListBoxStruct,8> | end | end__Q23gif28vector<16GM_ListBoxStruct,8>Fv |
| 0x002cd440 | 8 | 13 | gif::vector<16GM_ListBoxStruct,8> | begin | begin__Q23gif28vector<16GM_ListBoxStruct,8>Fv |
| 0x002cd450 | 140 | 13 | GM_ListBox | Key | Key__10GM_ListBoxFUiUi |
| 0x002cd4e0 | 96 | 13 | GM_ListCars | {dtor} | __dt__11GM_ListCarsFv |
| 0x002cd540 | 8 | 13 | GM_ListCars | SetPointer | SetPointer__11GM_ListCarsFPv |
| 0x002cd550 | 12 | 13 | GM_ListCars | SetDword | SetDword__11GM_ListCarsFUi |
| 0x002cd560 | 8 | 13 | GM_ListCars | GetDword | GetDword__11GM_ListCarsFv |
| 0x002cd570 | 216 | 13 | GM_ListCars | Tick | Tick__11GM_ListCarsFUi |
| 0x002cd650 | 512 | 13 | GM_ListCars | Draw | Draw__11GM_ListCarsFUi |
| 0x002cd850 | 36 | 13 | gif::vector<PUc,8> | size | size__Q23gif13vector<PUc,8>CFv |
| 0x002cd880 | 16 | 13 | gif::vector<PUc,8> | operator[] | __vc__Q23gif13vector<PUc,8>Fi |
| 0x002cd890 | 96 | 13 | GM_ViewScroll | {dtor} | __dt__13GM_ViewScrollFv |
| 0x002cd8f0 | 20 | 13 | GM_ViewScroll | GetRangeFloat | GetRangeFloat__13GM_ViewScrollFRfRf |
| 0x002cd910 | 8 | 13 | GM_Component | Draw | Draw__12GM_ComponentFUi |
| 0x002cd920 | 96 | 13 | GM_TextInput | {dtor} | __dt__12GM_TextInputFv |
| 0x002cd980 | 112 | 13 | GM_TextInput | Draw | Draw__12GM_TextInputFUi |
| 0x002cd9f0 | 96 | 13 | GM_StaticTextBig | {dtor} | __dt__16GM_StaticTextBigFv |
| 0x002cda50 | 60 | 13 | GM_StaticTextBig | SetText | SetText__16GM_StaticTextBigFPUc |
| 0x002cda90 | 76 | 13 | GM_StaticTextBig | SetTextID | SetTextID__16GM_StaticTextBigFUi |
| 0x002cdae0 | 56 | 13 | GM_StaticTextBig | SetDword | SetDword__16GM_StaticTextBigFUi |
| 0x002cdb20 | 60 | 13 | GM_StaticTextBig | GetFloat | GetFloat__16GM_StaticTextBigFv |
| 0x002cdb60 | 144 | 13 | GM_StaticTextBig | Draw | Draw__16GM_StaticTextBigFUi |
| 0x002cdbf0 | 96 | 13 | GM_StaticTextBig2Color | {dtor} | __dt__22GM_StaticTextBig2ColorFv |
| 0x002cdc50 | 60 | 13 | GM_StaticTextBig2Color | SetText | SetText__22GM_StaticTextBig2ColorFPUc |
| 0x002cdc90 | 76 | 13 | GM_StaticTextBig2Color | SetTextID | SetTextID__22GM_StaticTextBig2ColorFUi |
| 0x002cdce0 | 56 | 13 | GM_StaticTextBig2Color | SetDword | SetDword__22GM_StaticTextBig2ColorFUi |
| 0x002cdd20 | 60 | 13 | GM_StaticTextBig2Color | GetFloat | GetFloat__22GM_StaticTextBig2ColorFv |
| 0x002cdd60 | 144 | 13 | GM_StaticTextBig2Color | Draw | Draw__22GM_StaticTextBig2ColorFUi |
| 0x002cddf0 | 96 | 13 | GM_Karusel | {dtor} | __dt__10GM_KaruselFv |
| 0x002cde50 | 8 | 13 | GM_Karusel | SetPointer | SetPointer__10GM_KaruselFPv |
| 0x002cde60 | 68 | 13 | GM_Karusel | SetDword | SetDword__10GM_KaruselFUi |
| 0x002cdeb0 | 8 | 13 | GM_Karusel | GetDword | GetDword__10GM_KaruselFv |
| 0x002cdec0 | 28 | 13 | GM_Karusel | GetFloat | GetFloat__10GM_KaruselFv |
| 0x002cdee0 | 308 | 13 | GM_Karusel | Tick | Tick__10GM_KaruselFUi |
| 0x002ce020 | 476 | 13 | GM_Karusel | Draw | Draw__10GM_KaruselFUi |
| 0x002ce200 | 8 | 13 | std::vector<P8ITexture,Q23std21allocator<P8ITexture>> | operator[] | __vc__Q23std48vector<P8ITexture,Q23std21allocator<P8ITexture>>FUi |
| 0x002ce210 | 8 | 13 | std::__vector_imp<P8ITexture,Q23std21allocator<P8ITexture>,1> | operator[] | __vc__Q23std56__vector_imp<P8ITexture,Q23std21allocator<P8ITexture>,1>FUi |
| 0x002ce220 | 8 | 13 | std::vector<P8ITexture,Q23std21allocator<P8ITexture>> | size | size__Q23std48vector<P8ITexture,Q23std21allocator<P8ITexture>>CFv |
| 0x002ce230 | 8 | 13 | std::__vector_imp<P8ITexture,Q23std21allocator<P8ITexture>,1> | size | size__Q23std56__vector_imp<P8ITexture,Q23std21allocator<P8ITexture>,1>CFv |
| 0x002ce240 | 8 | 13 | std::__vector_pod<Ui,Q23std13allocator<Ui>> | size | size__Q23std38__vector_pod<Ui,Q23std13allocator<Ui>>CFv |
| 0x002ce250 | 356 | 13 | GM_Karusel | Key | Key__10GM_KaruselFUiUi |
| 0x002ce3c0 | 96 | 13 | GM_ColorSel | {dtor} | __dt__11GM_ColorSelFv |
| 0x002ce420 | 8 | 13 | GM_ColorSel | SetPointer | SetPointer__11GM_ColorSelFPv |
| 0x002ce430 | 68 | 13 | GM_ColorSel | SetDword | SetDword__11GM_ColorSelFUi |
| 0x002ce480 | 8 | 13 | GM_ColorSel | GetDword | GetDword__11GM_ColorSelFv |
| 0x002ce490 | 216 | 13 | GM_ColorSel | Tick | Tick__11GM_ColorSelFUi |
| 0x002ce570 | 360 | 13 | GM_ColorSel | Draw | Draw__11GM_ColorSelFUi |
| 0x002ce6e0 | 8 | 13 | std::vector<Ui,Q23std13allocator<Ui>> | operator[] | __vc__Q23std32vector<Ui,Q23std13allocator<Ui>>FUi |
| 0x002ce6f0 | 8 | 13 | std::vector<Ui,Q23std13allocator<Ui>> | size | size__Q23std32vector<Ui,Q23std13allocator<Ui>>CFv |
| 0x002ce700 | 332 | 13 | GM_ColorSel | Key | Key__11GM_ColorSelFUiUi |
| 0x002ce850 | 36 | GLOBAL |  | g_InitCompsm__Fv | g_InitCompsm__Fv |
| 0x002ce880 | 72 | 13 | G_Factory<14GM_MultiButton> | Make | Make__27G_Factory<14GM_MultiButton>FP15GM_DesignStructP12GM_Component |
| 0x002ce8d0 | 56 | 13 | G_Factory<14GM_MultiButton> | MakeDup | MakeDup__27G_Factory<14GM_MultiButton>FP12GM_Component |
| 0x002ce910 | 72 | 13 | G_Factory<10GM_WindowM> | Make | Make__23G_Factory<10GM_WindowM>FP15GM_DesignStructP12GM_Component |
| 0x002ce960 | 56 | 13 | G_Factory<10GM_WindowM> | MakeDup | MakeDup__23G_Factory<10GM_WindowM>FP12GM_Component |
| 0x002ce9a0 | 52 | 13 | GM_WindowM | {ctor} | __ct__10GM_WindowMFP12GM_Component |
| 0x002ce9e0 | 52 | 13 | GM_WindowM | {ctor} | __ct__10GM_WindowMFP15GM_DesignStructP12GM_Component |
| 0x002cea20 | 60 | 13 | GM_MultiButton | {ctor} | __ct__14GM_MultiButtonFP12GM_Component |
| 0x002cea60 | 172 | 13 | GM_MultiButton | {ctor} | __ct__14GM_MultiButtonFP15GM_DesignStructP12GM_Component |
| 0x002ceb10 | 96 | 13 | GM_MultiButton | {dtor} | __dt__14GM_MultiButtonFv |
| 0x002ceb70 | 12 | 13 | GM_MultiButton | GetType | GetType__14GM_MultiButtonFv |
| 0x002ceb80 | 152 | 13 | GM_MultiButton | IsOverActive | IsOverActive__14GM_MultiButtonFff |
| 0x002cec20 | 544 | 13 | GM_MultiButton | Draw | Draw__14GM_MultiButtonFUi |
| 0x002cee40 | 64 | 13 | GM_MultiButton | Key | Key__14GM_MultiButtonFUiUi |
| 0x002cee80 | 144 | 13 | GM_MultiButton | SetText | SetText__14GM_MultiButtonFPUc |
| 0x002cef10 | 84 | 13 | G_Factory<10GM_WindowM> | {ctor} | __ct__23G_Factory<10GM_WindowM>Fv |
| 0x002cef70 | 12 | 13 | GM_WindowM | GetType | GetType__10GM_WindowMFv |
| 0x002cef80 | 84 | 13 | G_Factory<14GM_MultiButton> | {ctor} | __ct__27G_Factory<14GM_MultiButton>Fv |
| 0x002cefe0 | 96 | 13 | GM_WindowM | {dtor} | __dt__10GM_WindowMFv |
| 0x002cf040 | 8 | 13 | GM_WindowM | CanBeParent | CanBeParent__10GM_WindowMFv |
| 0x002cf050 | 44 | 13 | GM_WindowM | GetClientOrigin | GetClientOrigin__10GM_WindowMFRfRf |
| 0x002cf080 | 612 | 13 | GM_WindowM | Draw | Draw__10GM_WindowMFUi |
| 0x002cf2f0 | 80 | GLOBAL | GM_MainMenuExit | OnClick | OnClick__15GM_MainMenuExitFUi |
| 0x002cf340 | 260 | GLOBAL | GM_MainMenu | OnCreate | OnCreate__11GM_MainMenuFv |
| 0x002cf450 | 52 | GLOBAL | GM_MainMenu | OnChange | OnChange__11GM_MainMenuFP12GM_ComponentP12GM_Component |
| 0x002cf490 | 660 | GLOBAL | GM_MainMenu | OnClick | OnClick__11GM_MainMenuFUi |
| 0x002cf730 | 236 | GLOBAL | GM_MainMenu | OnCloseChild | OnCloseChild__11GM_MainMenuFP7GM_MenuUiUi |
| 0x002cf820 | 464 | GLOBAL | GM_MainMenu | OnTick | OnTick__11GM_MainMenuFUi |
| 0x002cf9f0 | 68 | GLOBAL | GM_MainMenu | OnKey | OnKey__11GM_MainMenuFUcUi |
| 0x002cfa40 | 128 | GLOBAL | GM_GameMenuExit | OnClick | OnClick__15GM_GameMenuExitFUi |
| 0x002cfac0 | 164 | GLOBAL | GM_GameMenu | OnCreate | OnCreate__11GM_GameMenuFv |
| 0x002cfb70 | 96 | GLOBAL | GM_GameMenu | OnKey | OnKey__11GM_GameMenuFUcUi |
| 0x002cfbd0 | 396 | GLOBAL | GM_GameMenu | OnClick | OnClick__11GM_GameMenuFUi |
| 0x002cfd60 | 316 | GLOBAL | GM_GameMenu | OnCloseChild | OnCloseChild__11GM_GameMenuFP7GM_MenuUiUi |
| 0x002cfea0 | 240 | GLOBAL | GM_GameMenu | OnTick | OnTick__11GM_GameMenuFUi |
| 0x002cff90 | 280 | GLOBAL | GM_Options | OnCreate | OnCreate__10GM_OptionsFv |
| 0x002d00b0 | 604 | GLOBAL | GM_Options | OnClick | OnClick__10GM_OptionsFUi |
| 0x002d0310 | 8 | GLOBAL | GM_Options | OnTick | OnTick__10GM_OptionsFUi |
| 0x002d0320 | 104 | GLOBAL | GM_OptionsControls | OnCreate | OnCreate__18GM_OptionsControlsFv |
| 0x002d0390 | 8 | GLOBAL | GM_OptionsControls | OnClick | OnClick__18GM_OptionsControlsFUi |
| 0x002d03a0 | 220 | GLOBAL | GM_OptionsControls | OnKey | OnKey__18GM_OptionsControlsFUcUi |
| 0x002d0480 | 476 | GLOBAL | GM_OptionsControls | SetUpTexts | SetUpTexts__18GM_OptionsControlsFv |
| 0x002d0660 | 52 | GLOBAL | GM_OptionsScreenSlide | OnCreate | OnCreate__21GM_OptionsScreenSlideFv |
| 0x002d06a0 | 464 | GLOBAL | GM_OptionsScreenSlide | OnKey | OnKey__21GM_OptionsScreenSlideFUcUi |
| 0x002d0870 | 276 | GLOBAL | GM_FreeRide | OnCreate | OnCreate__11GM_FreeRideFv |
| 0x002d0990 | 100 | GLOBAL | GM_FreeRide | OnChange | OnChange__11GM_FreeRideFP12GM_ComponentP12GM_Component |
| 0x002d0a00 | 160 | GLOBAL | GM_FreeRide | OnCloseChild | OnCloseChild__11GM_FreeRideFP7GM_MenuUiUi |
| 0x002d0aa0 | 476 | GLOBAL | GM_FreeRide | OnClick | OnClick__11GM_FreeRideFUi |
| 0x002d0c80 | 144 | GLOBAL | GM_QuestionBox | {ctor} | __ct__14GM_QuestionBoxFUiUib |
| 0x002d0d10 | 520 | GLOBAL | GM_QuestionBox | OnCreate | OnCreate__14GM_QuestionBoxFv |
| 0x002d0f20 | 128 | GLOBAL | GM_QuestionBox | OnTick | OnTick__14GM_QuestionBoxFUi |
| 0x002d0fa0 | 72 | GLOBAL | GM_QuestionBox | OnKey | OnKey__14GM_QuestionBoxFUcUi |
| 0x002d0ff0 | 132 | GLOBAL | GM_QuestionBox | OnClick | OnClick__14GM_QuestionBoxFUi |
| 0x002d1080 | 484 | GLOBAL | GM_LoadGame | OnCreate | OnCreate__11GM_LoadGameFv |
| 0x002d1270 | 8 | 13 | std::__vector_imp<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>,0> | begin | begin__Q23std78__vector_imp<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>,0>Fv |
| 0x002d1280 | 28 | 13 | std::__vector_imp<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>,0> | end | end__Q23std78__vector_imp<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>,0>Fv |
| 0x002d12a0 | 152 | GLOBAL | GM_LoadGame | IsTexturePreloaded | IsTexturePreloaded__11GM_LoadGameFPc |
| 0x002d1340 | 376 | GLOBAL | GM_LoadGame | PreloadTextures | PreloadTextures__11GM_LoadGameFPQ23std70vector<19G_SaveGame_FileHead,Q23std32allocator<19G_SaveGame_FileHead>> |
| 0x002d14c0 | 96 | GLOBAL | GM_LoadGame | ReleasePreloadedTextures | ReleasePreloadedTextures__11GM_LoadGameFv |
| 0x002d1520 | 56 | GLOBAL | GM_LoadGame | OnClose | OnClose__11GM_LoadGameFv |
| 0x002d1560 | 456 | GLOBAL | GM_LoadGame | OnTick | OnTick__11GM_LoadGameFUi |
| 0x002d1730 | 204 | GLOBAL | GM_LoadGame | OnKey | OnKey__11GM_LoadGameFUcUi |
| 0x002d1800 | 296 | GLOBAL | GM_LoadGame | OnMouse | OnMouse__11GM_LoadGameFiiiUiUi |
| 0x002d1930 | 48 | GLOBAL | GM_LoadGame | OnChange | OnChange__11GM_LoadGameFP12GM_ComponentP12GM_Component |
| 0x002d1960 | 140 | GLOBAL | GM_LoadGame | OnCloseChild | OnCloseChild__11GM_LoadGameFP7GM_MenuUiUi |
| 0x002d19f0 | 188 | GLOBAL | GM_LoadGame | OnClick | OnClick__11GM_LoadGameFUi |
| 0x002d1ab0 | 244 | GLOBAL | GM_GameFailed | OnCreate | OnCreate__13GM_GameFailedFv |
| 0x002d1bb0 | 168 | GLOBAL | GM_GameFailed | OnClick | OnClick__13GM_GameFailedFUi |
| 0x002d1c60 | 1724 | GLOBAL | GM_ItemPickUp | OnCreate | OnCreate__13GM_ItemPickUpFv |
| 0x002d2320 | 260 | 13 | gif::vector<P10S_GameItem,8> | reserve | reserve__Q23gif23vector<P10S_GameItem,8>Fi |
| 0x002d2430 | 412 | GLOBAL | GM_ItemPickUp | OnKey | OnKey__13GM_ItemPickUpFUcUi |
| 0x002d25d0 | 228 | GLOBAL | GM_ItemPickUp | OnClick | OnClick__13GM_ItemPickUpFUi |
| 0x002d26c0 | 796 | GLOBAL | GM_ItemPickUp | PickUp | PickUp__13GM_ItemPickUpFUib |
| 0x002d29e0 | 320 | GLOBAL | GM_GunShop | OnCreate | OnCreate__10GM_GunShopFv |
| 0x002d2b20 | 76 | GLOBAL | GM_GunShop | OnClick | OnClick__10GM_GunShopFUi |
| 0x002d2b70 | 16 | GLOBAL | GM_GunShop | RefreshInventory | RefreshInventory__10GM_GunShopFv |
| 0x002d2b80 | 212 | GLOBAL | GM_LoadProgress | {ctor} | __ct__15GM_LoadProgressFUifPUc |
| 0x002d2c60 | 44 | GLOBAL | GM_LoadProgress | Destroy | Destroy__15GM_LoadProgressFv |
| 0x002d2c90 | 36 | GLOBAL | GM_LoadProgress | OnBeforeCreate | OnBeforeCreate__15GM_LoadProgressFv |
| 0x002d2cc0 | 816 | GLOBAL | GM_LoadProgress | OnCreate | OnCreate__15GM_LoadProgressFv |
| 0x002d2ff0 | 8 | GLOBAL | GM_LoadProgress | OnTick | OnTick__15GM_LoadProgressFUi |
| 0x002d3000 | 48 | GLOBAL | GM_LoadProgress | OnKey | OnKey__15GM_LoadProgressFUcUi |
| 0x002d3030 | 32 | GLOBAL | GM_LoadProgress | SetProgressGameInit | SetProgressGameInit__15GM_LoadProgressFf |
| 0x002d3050 | 12 | GLOBAL | GM_LoadProgress | SetProgressMissionOpen | SetProgressMissionOpen__15GM_LoadProgressFf |
| 0x002d3060 | 8 | GLOBAL | GM_LoadProgress | SetRawProgress | SetRawProgress__15GM_LoadProgressFf |
| 0x002d3070 | 160 | GLOBAL | GM_LoadProgress | Present | Present__15GM_LoadProgressFv |
| 0x002d3110 | 96 | GLOBAL | GM_PressStart | OnCreate | OnCreate__13GM_PressStartFv |
| 0x002d3170 | 244 | GLOBAL | GM_PressStart | OnTick | OnTick__13GM_PressStartFUi |
| 0x002d3270 | 96 | GLOBAL | GM_PressStart | OnKey | OnKey__13GM_PressStartFUcUi |
| 0x002d32d0 | 104 | GLOBAL | GM_GameMap | OnCreate | OnCreate__10GM_GameMapFv |
| 0x002d3340 | 68 | GLOBAL | GM_GameMap | OnKey | OnKey__10GM_GameMapFUcUi |
| 0x002d3390 | 8 | GLOBAL | GM_GameMap | OnClick | OnClick__10GM_GameMapFUi |
| 0x002d33a0 | 716 | GLOBAL | GM_GameMap | OnTick | OnTick__10GM_GameMapFUi |
| 0x002d3670 | 388 | GLOBAL | GM_TutorialMessage | OnCreate | OnCreate__18GM_TutorialMessageFv |
| 0x002d3800 | 52 | GLOBAL | GM_TutorialMessage | OnKey | OnKey__18GM_TutorialMessageFUcUi |
| 0x002d3840 | 36 | GLOBAL | GM_HotRodMessage | OnCreate | OnCreate__16GM_HotRodMessageFv |
| 0x002d3870 | 108 | GLOBAL | GM_HotRodMessage | OnTick | OnTick__16GM_HotRodMessageFUi |
| 0x002d38e0 | 52 | GLOBAL | GM_HotRodMessage | OnKey | OnKey__16GM_HotRodMessageFUcUi |
| 0x002d3920 | 96 | 13 | GM_HotRodMessage | {dtor} | __dt__16GM_HotRodMessageFv |
| 0x002d3980 | 12 | 13 | GM_HotRodMessage | OnBeforeCreate | OnBeforeCreate__16GM_HotRodMessageFv |
| 0x002d3990 | 8 | 13 | GM_Menu | OnClose | OnClose__7GM_MenuFv |
| 0x002d39a0 | 96 | 13 | GM_TutorialMessage | {dtor} | __dt__18GM_TutorialMessageFv |
| 0x002d3a00 | 12 | 13 | GM_TutorialMessage | OnBeforeCreate | OnBeforeCreate__18GM_TutorialMessageFv |
| 0x002d3a10 | 96 | 13 | GM_GameMap | {dtor} | __dt__10GM_GameMapFv |
| 0x002d3a70 | 12 | 13 | GM_GameMap | OnBeforeCreate | OnBeforeCreate__10GM_GameMapFv |
| 0x002d3a80 | 96 | 13 | GM_PressStart | {dtor} | __dt__13GM_PressStartFv |
| 0x002d3ae0 | 12 | 13 | GM_PressStart | OnBeforeCreate | OnBeforeCreate__13GM_PressStartFv |
| 0x002d3af0 | 96 | 13 | GM_GunShop | {dtor} | __dt__10GM_GunShopFv |
| 0x002d3b50 | 12 | 13 | GM_GunShop | OnBeforeCreate | OnBeforeCreate__10GM_GunShopFv |
| 0x002d3b60 | 8 | 13 | GM_GunShop | OnKey | OnKey__10GM_GunShopFUcUi |
| 0x002d3b70 | 140 | 13 | GM_ItemPickUp | {dtor} | __dt__13GM_ItemPickUpFv |
| 0x002d3c00 | 12 | 13 | GM_ItemPickUp | OnBeforeCreate | OnBeforeCreate__13GM_ItemPickUpFv |
| 0x002d3c10 | 96 | 13 | GM_GameFailed | {dtor} | __dt__13GM_GameFailedFv |
| 0x002d3c70 | 12 | 13 | GM_GameFailed | OnBeforeCreate | OnBeforeCreate__13GM_GameFailedFv |
| 0x002d3c80 | 8 | 13 | GM_GameFailed | OnKey | OnKey__13GM_GameFailedFUcUi |
| 0x002d3c90 | 124 | 13 | GM_LoadGame | {dtor} | __dt__11GM_LoadGameFv |
| 0x002d3d10 | 12 | 13 | GM_LoadGame | OnBeforeCreate | OnBeforeCreate__11GM_LoadGameFv |
| 0x002d3d20 | 96 | 13 | GM_QuestionBox | {dtor} | __dt__14GM_QuestionBoxFv |
| 0x002d3d80 | 72 | 13 | GM_QuestionBox | OnBeforeCreate | OnBeforeCreate__14GM_QuestionBoxFv |
| 0x002d3dd0 | 96 | 13 | GM_FreeRide | {dtor} | __dt__11GM_FreeRideFv |
| 0x002d3e30 | 12 | 13 | GM_FreeRide | OnBeforeCreate | OnBeforeCreate__11GM_FreeRideFv |
| 0x002d3e40 | 96 | 13 | GM_OptionsScreenSlide | {dtor} | __dt__21GM_OptionsScreenSlideFv |
| 0x002d3ea0 | 12 | 13 | GM_OptionsScreenSlide | OnBeforeCreate | OnBeforeCreate__21GM_OptionsScreenSlideFv |
| 0x002d3eb0 | 96 | 13 | GM_OptionsControls | {dtor} | __dt__18GM_OptionsControlsFv |
| 0x002d3f10 | 12 | 13 | GM_OptionsControls | OnBeforeCreate | OnBeforeCreate__18GM_OptionsControlsFv |
| 0x002d3f20 | 96 | 13 | GM_Options | {dtor} | __dt__10GM_OptionsFv |
| 0x002d3f80 | 12 | 13 | GM_Options | OnBeforeCreate | OnBeforeCreate__10GM_OptionsFv |
| 0x002d3f90 | 8 | 13 | GM_Options | OnKey | OnKey__10GM_OptionsFUcUi |
| 0x002d3fa0 | 96 | 13 | GM_GameMenu | {dtor} | __dt__11GM_GameMenuFv |
| 0x002d4000 | 20 | 13 | GM_GameMenu | OnBeforeCreate | OnBeforeCreate__11GM_GameMenuFv |
| 0x002d4020 | 96 | 13 | GM_GameMenuExit | {dtor} | __dt__15GM_GameMenuExitFv |
| 0x002d4080 | 12 | 13 | GM_GameMenuExit | OnBeforeCreate | OnBeforeCreate__15GM_GameMenuExitFv |
| 0x002d4090 | 60 | 13 | GM_GameMenuExit | OnCreate | OnCreate__15GM_GameMenuExitFv |
| 0x002d40d0 | 96 | 13 | GM_MainMenu | {dtor} | __dt__11GM_MainMenuFv |
| 0x002d4130 | 12 | 13 | GM_MainMenu | OnBeforeCreate | OnBeforeCreate__11GM_MainMenuFv |
| 0x002d4140 | 96 | 13 | GM_MainMenuExit | {dtor} | __dt__15GM_MainMenuExitFv |
| 0x002d41a0 | 12 | 13 | GM_MainMenuExit | OnBeforeCreate | OnBeforeCreate__15GM_MainMenuExitFv |
| 0x002d41b0 | 36 | 13 | GM_MainMenuExit | OnCreate | OnCreate__15GM_MainMenuExitFv |
| 0x002d41e0 | 108 | GLOBAL | GM_PSX_Checkmemorycard | OnCreate | OnCreate__22GM_PSX_CheckmemorycardFv |
| 0x002d4250 | 404 | GLOBAL | GM_PSX_Checkmemorycard | OnTick | OnTick__22GM_PSX_CheckmemorycardFUi |
| 0x002d43f0 | 576 | GLOBAL | GM_PSX_Nomemorycard | OnCreate | OnCreate__19GM_PSX_NomemorycardFv |
| 0x002d4630 | 8 | GLOBAL | GM_PSX_Nomemorycard | OnKey | OnKey__19GM_PSX_NomemorycardFUcUi |
| 0x002d4640 | 156 | GLOBAL | GM_PSX_Nomemorycard | OnClick | OnClick__19GM_PSX_NomemorycardFUi |
| 0x002d46e0 | 8 | GLOBAL | GM_PSX_Nomemorycard | OnTick | OnTick__19GM_PSX_NomemorycardFUi |
| 0x002d46f0 | 1160 | GLOBAL | GM_PSX_Selectmemorycard | OnCreate | OnCreate__23GM_PSX_SelectmemorycardFv |
| 0x002d4b80 | 260 | 13 | gif::vector<P12GM_Component,8> | reserve | reserve__Q23gif25vector<P12GM_Component,8>Fi |
| 0x002d4c90 | 116 | GLOBAL | GM_PSX_Selectmemorycard | OnTick | OnTick__23GM_PSX_SelectmemorycardFUi |
| 0x002d4d10 | 108 | GLOBAL | GM_PSX_Selectmemorycard | OnClick | OnClick__23GM_PSX_SelectmemorycardFUi |
| 0x002d4d80 | 84 | GLOBAL | GM_PSX_Selectmemorycard | OnKey | OnKey__23GM_PSX_SelectmemorycardFUcUi |
| 0x002d4de0 | 556 | GLOBAL | GM_PSX_Formatmemorycard | OnCreate | OnCreate__23GM_PSX_FormatmemorycardFv |
| 0x002d5010 | 116 | GLOBAL | GM_PSX_Formatmemorycard | OnTick | OnTick__23GM_PSX_FormatmemorycardFUi |
| 0x002d5090 | 228 | GLOBAL | GM_PSX_Formatmemorycard | OnClick | OnClick__23GM_PSX_FormatmemorycardFUi |
| 0x002d5180 | 8 | GLOBAL | GM_PSX_Formatmemorycard | OnKey | OnKey__23GM_PSX_FormatmemorycardFUcUi |
| 0x002d5190 | 92 | GLOBAL | GM_PSX_Formatingmemorycard | OnCreate | OnCreate__26GM_PSX_FormatingmemorycardFv |
| 0x002d51f0 | 404 | GLOBAL | GM_PSX_Formatingmemorycard | OnTick | OnTick__26GM_PSX_FormatingmemorycardFUi |
| 0x002d5390 | 40 | GLOBAL | GM_PSX_Formatdone | OnCreate | OnCreate__17GM_PSX_FormatdoneFv |
| 0x002d53c0 | 192 | GLOBAL | GM_PSX_Formatdone | OnTick | OnTick__17GM_PSX_FormatdoneFUi |
| 0x002d5480 | 80 | GLOBAL | GM_PSX_Formatdone | OnClick | OnClick__17GM_PSX_FormatdoneFUi |
| 0x002d54d0 | 8 | GLOBAL | GM_PSX_Formatdone | OnKey | OnKey__17GM_PSX_FormatdoneFUcUi |
| 0x002d54e0 | 36 | GLOBAL | GM_PSX_Formatdonefail | OnCreate | OnCreate__21GM_PSX_FormatdonefailFv |
| 0x002d5510 | 8 | GLOBAL | GM_PSX_Formatdonefail | OnTick | OnTick__21GM_PSX_FormatdonefailFUi |
| 0x002d5520 | 156 | GLOBAL | GM_PSX_Formatdonefail | OnClick | OnClick__21GM_PSX_FormatdonefailFUi |
| 0x002d55c0 | 92 | GLOBAL | GM_PSX_Formatdonefail | OnKey | OnKey__21GM_PSX_FormatdonefailFUcUi |
| 0x002d5620 | 712 | GLOBAL | GM_PSX_Notenoughmemory | OnCreate | OnCreate__22GM_PSX_NotenoughmemoryFv |
| 0x002d58f0 | 116 | GLOBAL | GM_PSX_Notenoughmemory | OnTick | OnTick__22GM_PSX_NotenoughmemoryFUi |
| 0x002d5970 | 156 | GLOBAL | GM_PSX_Notenoughmemory | OnClick | OnClick__22GM_PSX_NotenoughmemoryFUi |
| 0x002d5a10 | 8 | GLOBAL | GM_PSX_Notenoughmemory | OnKey | OnKey__22GM_PSX_NotenoughmemoryFUcUi |
| 0x002d5a20 | 228 | GLOBAL | GM_PSX_Process | OnCreate | OnCreate__14GM_PSX_ProcessFv |
| 0x002d5b10 | 788 | GLOBAL | GM_PSX_Process | OnTick | OnTick__14GM_PSX_ProcessFUi |
| 0x002d5e30 | 188 | GLOBAL | GM_PSX_ProcessDone | OnCreate | OnCreate__18GM_PSX_ProcessDoneFv |
| 0x002d5ef0 | 260 | GLOBAL | GM_PSX_ProcessDone | OnTick | OnTick__18GM_PSX_ProcessDoneFUi |
| 0x002d6000 | 8 | GLOBAL | GM_PSX_ProcessDone | OnKey | OnKey__18GM_PSX_ProcessDoneFUcUi |
| 0x002d6010 | 240 | GLOBAL | GM_PSX_ProcessDone | OnClick | OnClick__18GM_PSX_ProcessDoneFUi |
| 0x002d6100 | 280 | GLOBAL | GM_PSX_ProcessDoneFail | OnCreate | OnCreate__22GM_PSX_ProcessDoneFailFv |
| 0x002d6220 | 8 | GLOBAL | GM_PSX_ProcessDoneFail | OnTick | OnTick__22GM_PSX_ProcessDoneFailFUi |
| 0x002d6230 | 8 | GLOBAL | GM_PSX_ProcessDoneFail | OnKey | OnKey__22GM_PSX_ProcessDoneFailFUcUi |
| 0x002d6240 | 600 | GLOBAL | GM_PSX_ProcessDoneFail | OnClick | OnClick__22GM_PSX_ProcessDoneFailFUi |
| 0x002d64a0 | 252 | GLOBAL | GM_PSX_Hlaska | OnCreate | OnCreate__13GM_PSX_HlaskaFv |
| 0x002d65a0 | 116 | GLOBAL | GM_PSX_Hlaska | OnTick | OnTick__13GM_PSX_HlaskaFUi |
| 0x002d6620 | 84 | GLOBAL | GM_PSX_Hlaska | OnKey | OnKey__13GM_PSX_HlaskaFUcUi |
| 0x002d6680 | 116 | GLOBAL | GM_PSX_CardReplaced | OnTick | OnTick__19GM_PSX_CardReplacedFUi |
| 0x002d6700 | 156 | GLOBAL | GM_PSX_CardReplaced | OnClick | OnClick__19GM_PSX_CardReplacedFUi |
| 0x002d67a0 | 56 | GLOBAL | GM_PSX_CautionAutosave | OnKey | OnKey__22GM_PSX_CautionAutosaveFUcUi |
| 0x002d67e0 | 112 | GLOBAL | GM_PSX_CautionAutosave | OnCreate | OnCreate__22GM_PSX_CautionAutosaveFv |
| 0x002d6850 | 64 | GLOBAL | GM_PSX_CautionAutosave | OnTick | OnTick__22GM_PSX_CautionAutosaveFUi |
| 0x002d6890 | 100 | GLOBAL | GM_PSX_MasterMenu | {ctor} | __ct__17GM_PSX_MasterMenuF13E_PSX_PROCESS |
| 0x002d6900 | 24 | GLOBAL | GM_PSX_MasterMenu | OnCreate | OnCreate__17GM_PSX_MasterMenuFv |
| 0x002d6920 | 8 | GLOBAL | GM_PSX_MasterMenu | OnClose | OnClose__17GM_PSX_MasterMenuFv |
| 0x002d6930 | 7076 | GLOBAL | GM_PSX_MasterMenu | OnTick | OnTick__17GM_PSX_MasterMenuFUi |
| 0x002d84e0 | 344 | GLOBAL | GM_PSX_MasterMenu | consistentRetry | consistentRetry__17GM_PSX_MasterMenuFv |
| 0x002d8640 | 2524 | GLOBAL | GM_PSX_MasterMenu | OnCloseChild | OnCloseChild__17GM_PSX_MasterMenuFP7GM_MenuUiUi |
| 0x002d9020 | 96 | 13 | GM_PSX_MasterMenu | {dtor} | __dt__17GM_PSX_MasterMenuFv |
| 0x002d9080 | 48 | 13 | GM_PSX_MasterMenu | OnBeforeCreate | OnBeforeCreate__17GM_PSX_MasterMenuFv |
| 0x002d90b0 | 8 | 13 | GM_PSX_MasterMenu | OnKey | OnKey__17GM_PSX_MasterMenuFUcUi |
| 0x002d90c0 | 96 | 13 | GM_PSX_CautionAutosave | {dtor} | __dt__22GM_PSX_CautionAutosaveFv |
| 0x002d9120 | 48 | 13 | GM_PSX_CautionAutosave | OnBeforeCreate | OnBeforeCreate__22GM_PSX_CautionAutosaveFv |
| 0x002d9150 | 96 | 13 | GM_PSX_CardReplaced | {dtor} | __dt__19GM_PSX_CardReplacedFv |
| 0x002d91b0 | 48 | 13 | GM_PSX_CardReplaced | OnBeforeCreate | OnBeforeCreate__19GM_PSX_CardReplacedFv |
| 0x002d91e0 | 8 | 13 | GM_PSX_CardReplaced | OnKey | OnKey__19GM_PSX_CardReplacedFUcUi |
| 0x002d91f0 | 96 | 13 | GM_PSX_Hlaska | {dtor} | __dt__13GM_PSX_HlaskaFv |
| 0x002d9250 | 48 | 13 | GM_PSX_Hlaska | OnBeforeCreate | OnBeforeCreate__13GM_PSX_HlaskaFv |
| 0x002d9280 | 96 | 13 | GM_PSX_ProcessDoneFail | {dtor} | __dt__22GM_PSX_ProcessDoneFailFv |
| 0x002d92e0 | 48 | 13 | GM_PSX_ProcessDoneFail | OnBeforeCreate | OnBeforeCreate__22GM_PSX_ProcessDoneFailFv |
| 0x002d9310 | 96 | 13 | GM_PSX_ProcessDone | {dtor} | __dt__18GM_PSX_ProcessDoneFv |
| 0x002d9370 | 48 | 13 | GM_PSX_ProcessDone | OnBeforeCreate | OnBeforeCreate__18GM_PSX_ProcessDoneFv |
| 0x002d93a0 | 96 | 13 | GM_PSX_Process | {dtor} | __dt__14GM_PSX_ProcessFv |
| 0x002d9400 | 48 | 13 | GM_PSX_Process | OnBeforeCreate | OnBeforeCreate__14GM_PSX_ProcessFv |
| 0x002d9430 | 8 | 13 | GM_PSX_Process | OnKey | OnKey__14GM_PSX_ProcessFUcUi |
| 0x002d9440 | 96 | 13 | GM_PSX_Notenoughmemory | {dtor} | __dt__22GM_PSX_NotenoughmemoryFv |
| 0x002d94a0 | 48 | 13 | GM_PSX_Notenoughmemory | OnBeforeCreate | OnBeforeCreate__22GM_PSX_NotenoughmemoryFv |
| 0x002d94d0 | 96 | 13 | GM_PSX_Formatdonefail | {dtor} | __dt__21GM_PSX_FormatdonefailFv |
| 0x002d9530 | 48 | 13 | GM_PSX_Formatdonefail | OnBeforeCreate | OnBeforeCreate__21GM_PSX_FormatdonefailFv |
| 0x002d9560 | 96 | 13 | GM_PSX_Formatdone | {dtor} | __dt__17GM_PSX_FormatdoneFv |
| 0x002d95c0 | 48 | 13 | GM_PSX_Formatdone | OnBeforeCreate | OnBeforeCreate__17GM_PSX_FormatdoneFv |
| 0x002d95f0 | 96 | 13 | GM_PSX_Formatingmemorycard | {dtor} | __dt__26GM_PSX_FormatingmemorycardFv |
| 0x002d9650 | 48 | 13 | GM_PSX_Formatingmemorycard | OnBeforeCreate | OnBeforeCreate__26GM_PSX_FormatingmemorycardFv |
| 0x002d9680 | 8 | 13 | GM_PSX_Formatingmemorycard | OnKey | OnKey__26GM_PSX_FormatingmemorycardFUcUi |
| 0x002d9690 | 96 | 13 | GM_PSX_Formatmemorycard | {dtor} | __dt__23GM_PSX_FormatmemorycardFv |
| 0x002d96f0 | 48 | 13 | GM_PSX_Formatmemorycard | OnBeforeCreate | OnBeforeCreate__23GM_PSX_FormatmemorycardFv |
| 0x002d9720 | 96 | 13 | GM_PSX_Selectmemorycard | {dtor} | __dt__23GM_PSX_SelectmemorycardFv |
| 0x002d9780 | 48 | 13 | GM_PSX_Selectmemorycard | OnBeforeCreate | OnBeforeCreate__23GM_PSX_SelectmemorycardFv |
| 0x002d97b0 | 96 | 13 | GM_PSX_Nomemorycard | {dtor} | __dt__19GM_PSX_NomemorycardFv |
| 0x002d9810 | 48 | 13 | GM_PSX_Nomemorycard | OnBeforeCreate | OnBeforeCreate__19GM_PSX_NomemorycardFv |
| 0x002d9840 | 96 | 13 | GM_PSX_Checkmemorycard | {dtor} | __dt__22GM_PSX_CheckmemorycardFv |
| 0x002d98a0 | 48 | 13 | GM_PSX_Checkmemorycard | OnBeforeCreate | OnBeforeCreate__22GM_PSX_CheckmemorycardFv |
| 0x002d98d0 | 8 | 13 | GM_PSX_Checkmemorycard | OnKey | OnKey__22GM_PSX_CheckmemorycardFUcUi |
| 0x002d98e0 | 280 | GLOBAL | GM_Racing | OnClick | OnClick__9GM_RacingFUi |
| 0x002d9a00 | 36 | GLOBAL | GM_Racing | OnChange | OnChange__9GM_RacingFP12GM_ComponentP12GM_Component |
| 0x002d9a30 | 220 | GLOBAL | GM_Racing | OpenMenuForSingleRace | OpenMenuForSingleRace__9GM_RacingFb |
| 0x002d9b10 | 468 | GLOBAL | GM_Racing | OpenMenuForChampionship | OpenMenuForChampionship__9GM_RacingFi |
| 0x002d9cf0 | 8 | GLOBAL | GM_Racing | OnTick | OnTick__9GM_RacingFUi |
| 0x002d9d00 | 344 | GLOBAL | GM_SingleRace | OnCreate | OnCreate__13GM_SingleRaceFv |
| 0x002d9e60 | 460 | GLOBAL | GM_SingleRace | OnClick | OnClick__13GM_SingleRaceFUi |
| 0x002da030 | 52 | GLOBAL | GM_SingleRace | OnKey | OnKey__13GM_SingleRaceFUcUi |
| 0x002da070 | 52 | GLOBAL | GM_SingleRace | OnTick | OnTick__13GM_SingleRaceFUi |
| 0x002da0b0 | 332 | GLOBAL | GM_SingleRace | OnCloseChild | OnCloseChild__13GM_SingleRaceFP7GM_MenuUiUi |
| 0x002da200 | 400 | GLOBAL | GM_SelectCircuit | OnCreate | OnCreate__16GM_SelectCircuitFv |
| 0x002da390 | 136 | GLOBAL | GM_SelectCircuit | OnClick | OnClick__16GM_SelectCircuitFUi |
| 0x002da420 | 120 | GLOBAL | GM_SelectCircuit | OnChange | OnChange__16GM_SelectCircuitFP12GM_ComponentP12GM_Component |
| 0x002da4a0 | 644 | GLOBAL | GM_SelectCarManufacturer | OnCreate | OnCreate__24GM_SelectCarManufacturerFv |
| 0x002da730 | 200 | GLOBAL | GM_SelectCarManufacturer | OnClick | OnClick__24GM_SelectCarManufacturerFUi |
| 0x002da800 | 68 | GLOBAL | GM_SelectCarManufacturer | OnCloseChild | OnCloseChild__24GM_SelectCarManufacturerFP7GM_MenuUiUi |
| 0x002da850 | 752 | GLOBAL | GM_SelectCarModel | OnCreate | OnCreate__17GM_SelectCarModelFv |
| 0x002dab40 | 36 | GLOBAL | GM_SelectCarModel | OnClose | OnClose__17GM_SelectCarModelFv |
| 0x002dab70 | 68 | GLOBAL | GM_SelectCarModel | OnCloseChild | OnCloseChild__17GM_SelectCarModelFP7GM_MenuUiUi |
| 0x002dabc0 | 344 | GLOBAL | GM_SelectCarModel | OnClick | OnClick__17GM_SelectCarModelFUi |
| 0x002dad20 | 332 | GLOBAL | GM_SelectCarModel | OnChange | OnChange__17GM_SelectCarModelFP12GM_ComponentP12GM_Component |
| 0x002dae70 | 244 | GLOBAL | GM_SelectCarModification | GetCarColor | GetCarColor__24GM_SelectCarModificationFUiUi |
| 0x002daf70 | 176 | GLOBAL | GM_SelectCarModification | OnCreate | OnCreate__24GM_SelectCarModificationFv |
| 0x002db020 | 80 | GLOBAL | GM_SelectCarModification | OnTick | OnTick__24GM_SelectCarModificationFUi |
| 0x002db070 | 144 | GLOBAL | GM_SelectCarModification | OnKey | OnKey__24GM_SelectCarModificationFUcUi |
| 0x002db100 | 8 | GLOBAL | GM_SelectCarModification | OnClick | OnClick__24GM_SelectCarModificationFUi |
| 0x002db110 | 268 | GLOBAL | GM_SelectOpponents | OnCreate | OnCreate__18GM_SelectOpponentsFv |
| 0x002db220 | 112 | GLOBAL | GM_SelectOpponents | OnCloseChild | OnCloseChild__18GM_SelectOpponentsFP7GM_MenuUiUi |
| 0x002db290 | 52 | GLOBAL | GM_SelectOpponents | OnKey | OnKey__18GM_SelectOpponentsFUcUi |
| 0x002db2d0 | 248 | GLOBAL | GM_SelectOpponents | OnClick | OnClick__18GM_SelectOpponentsFUi |
| 0x002db3d0 | 1616 | GLOBAL | GM_SingleRaceSecond | OnCreate | OnCreate__19GM_SingleRaceSecondFv |
| 0x002dba20 | 168 | GLOBAL | GM_SingleRaceSecond | OnClick | OnClick__19GM_SingleRaceSecondFUi |
| 0x002dbad0 | 3196 | GLOBAL | GM_SingleRaceResults | OnCreate | OnCreate__20GM_SingleRaceResultsFv |
| 0x002dc750 | 252 | GLOBAL | GM_SingleRaceResults | OnClick | OnClick__20GM_SingleRaceResultsFUi |
| 0x002dc850 | 208 | GLOBAL | GM_SingleRaceResults | OnKey | OnKey__20GM_SingleRaceResultsFUcUi |
| 0x002dc920 | 84 | GLOBAL | GM_SingleRaceResults | OnTick | OnTick__20GM_SingleRaceResultsFUi |
| 0x002dc980 | 144 | GLOBAL | GM_DamageDifficulty | OnCreate | OnCreate__19GM_DamageDifficultyFv |
| 0x002dca10 | 160 | GLOBAL | GM_DamageDifficulty | OnClick | OnClick__19GM_DamageDifficultyFUi |
| 0x002dcab0 | 64 | GLOBAL | GM_DamageDifficulty | OnKey | OnKey__19GM_DamageDifficultyFUcUi |
| 0x002dcaf0 | 904 | GLOBAL | GM_Championship | OnCreate | OnCreate__15GM_ChampionshipFv |
| 0x002dce80 | 232 | GLOBAL | GM_Championship | OnClick | OnClick__15GM_ChampionshipFUi |
| 0x002dcf70 | 52 | GLOBAL | GM_Championship | OnTick | OnTick__15GM_ChampionshipFUi |
| 0x002dcfb0 | 632 | GLOBAL | GM_SelectInRound | OnCreate | OnCreate__16GM_SelectInRoundFv |
| 0x002dd230 | 188 | GLOBAL | GM_SelectInRound | OnClick | OnClick__16GM_SelectInRoundFUi |
| 0x002dd2f0 | 52 | GLOBAL | GM_SelectInRound | OnTick | OnTick__16GM_SelectInRoundFUi |
| 0x002dd330 | 1896 | GLOBAL | GM_ChampionshipRace | OnCreate | OnCreate__19GM_ChampionshipRaceFv |
| 0x002ddaa0 | 320 | GLOBAL | GM_ChampionshipRace | OnClick | OnClick__19GM_ChampionshipRaceFUi |
| 0x002ddbe0 | 68 | GLOBAL | GM_ChampionshipRace | OnCloseChild | OnCloseChild__19GM_ChampionshipRaceFP7GM_MenuUiUi |
| 0x002ddc30 | 308 | GLOBAL | GM_GameMenuRacing | OnClick | OnClick__17GM_GameMenuRacingFUi |
| 0x002ddd70 | 132 | GLOBAL | GM_GameMenuRacing | OnCloseChild | OnCloseChild__17GM_GameMenuRacingFP7GM_MenuUiUi |
| 0x002dde00 | 108 | GLOBAL | GM_GameMenuRacing | OnTick | OnTick__17GM_GameMenuRacingFUi |
| 0x002dde70 | 92 | GLOBAL | GM_GameMenuRacing | OnKey | OnKey__17GM_GameMenuRacingFUcUi |
| 0x002dded0 | 2552 | GLOBAL | GM_ChampionshipResults | OnCreate | OnCreate__22GM_ChampionshipResultsFv |
| 0x002de8d0 | 84 | GLOBAL | GM_ChampionshipResults | OnTick | OnTick__22GM_ChampionshipResultsFUi |
| 0x002de930 | 192 | GLOBAL | GM_ChampionshipResults | OnClick | OnClick__22GM_ChampionshipResultsFUi |
| 0x002de9f0 | 232 | GLOBAL | GM_ChampionshipResults | OnKey | OnKey__22GM_ChampionshipResultsFUcUi |
| 0x002deae0 | 380 | GLOBAL | GM_BonusCar | OnCreate | OnCreate__11GM_BonusCarFv |
| 0x002dec60 | 44 | GLOBAL | GM_BonusCar | OnClick | OnClick__11GM_BonusCarFUi |
| 0x002dec90 | 84 | GLOBAL | GM_BonusCar | OnKey | OnKey__11GM_BonusCarFUcUi |
| 0x002decf0 | 96 | 13 | GM_BonusCar | {dtor} | __dt__11GM_BonusCarFv |
| 0x002ded50 | 12 | 13 | GM_BonusCar | OnBeforeCreate | OnBeforeCreate__11GM_BonusCarFv |
| 0x002ded60 | 96 | 13 | GM_ChampionshipResults | {dtor} | __dt__22GM_ChampionshipResultsFv |
| 0x002dedc0 | 12 | 13 | GM_ChampionshipResults | OnBeforeCreate | OnBeforeCreate__22GM_ChampionshipResultsFv |
| 0x002dedd0 | 96 | 13 | GM_GameMenuRacing | {dtor} | __dt__17GM_GameMenuRacingFv |
| 0x002dee30 | 12 | 13 | GM_GameMenuRacing | OnBeforeCreate | OnBeforeCreate__17GM_GameMenuRacingFv |
| 0x002dee40 | 48 | 13 | GM_GameMenuRacing | OnCreate | OnCreate__17GM_GameMenuRacingFv |
| 0x002dee70 | 96 | 13 | GM_ChampionshipRace | {dtor} | __dt__19GM_ChampionshipRaceFv |
| 0x002deed0 | 32 | 13 | GM_ChampionshipRace | OnBeforeCreate | OnBeforeCreate__19GM_ChampionshipRaceFv |
| 0x002deef0 | 96 | 13 | GM_SelectInRound | {dtor} | __dt__16GM_SelectInRoundFv |
| 0x002def50 | 12 | 13 | GM_SelectInRound | OnBeforeCreate | OnBeforeCreate__16GM_SelectInRoundFv |
| 0x002def60 | 96 | 13 | GM_Championship | {dtor} | __dt__15GM_ChampionshipFv |
| 0x002defc0 | 12 | 13 | GM_Championship | OnBeforeCreate | OnBeforeCreate__15GM_ChampionshipFv |
| 0x002defd0 | 96 | 13 | GM_DamageDifficulty | {dtor} | __dt__19GM_DamageDifficultyFv |
| 0x002df030 | 12 | 13 | GM_DamageDifficulty | OnBeforeCreate | OnBeforeCreate__19GM_DamageDifficultyFv |
| 0x002df040 | 96 | 13 | GM_SingleRaceResults | {dtor} | __dt__20GM_SingleRaceResultsFv |
| 0x002df0a0 | 12 | 13 | GM_SingleRaceResults | OnBeforeCreate | OnBeforeCreate__20GM_SingleRaceResultsFv |
| 0x002df0b0 | 96 | 13 | GM_SingleRaceSecond | {dtor} | __dt__19GM_SingleRaceSecondFv |
| 0x002df110 | 32 | 13 | GM_SingleRaceSecond | OnBeforeCreate | OnBeforeCreate__19GM_SingleRaceSecondFv |
| 0x002df130 | 96 | 13 | GM_SelectOpponents | {dtor} | __dt__18GM_SelectOpponentsFv |
| 0x002df190 | 12 | 13 | GM_SelectOpponents | OnBeforeCreate | OnBeforeCreate__18GM_SelectOpponentsFv |
| 0x002df1a0 | 124 | 13 | GM_SelectCarModification | {dtor} | __dt__24GM_SelectCarModificationFv |
| 0x002df220 | 12 | 13 | GM_SelectCarModification | OnBeforeCreate | OnBeforeCreate__24GM_SelectCarModificationFv |
| 0x002df230 | 124 | 13 | GM_SelectCarModel | {dtor} | __dt__17GM_SelectCarModelFv |
| 0x002df2b0 | 12 | 13 | GM_SelectCarModel | OnBeforeCreate | OnBeforeCreate__17GM_SelectCarModelFv |
| 0x002df2c0 | 96 | 13 | GM_SelectCarManufacturer | {dtor} | __dt__24GM_SelectCarManufacturerFv |
| 0x002df320 | 32 | 13 | GM_SelectCarManufacturer | OnBeforeCreate | OnBeforeCreate__24GM_SelectCarManufacturerFv |
| 0x002df340 | 96 | 13 | GM_SelectCircuit | {dtor} | __dt__16GM_SelectCircuitFv |
| 0x002df3a0 | 32 | 13 | GM_SelectCircuit | OnBeforeCreate | OnBeforeCreate__16GM_SelectCircuitFv |
| 0x002df3c0 | 96 | 13 | GM_SingleRace | {dtor} | __dt__13GM_SingleRaceFv |
| 0x002df420 | 32 | 13 | GM_SingleRace | OnBeforeCreate | OnBeforeCreate__13GM_SingleRaceFv |
| 0x002df440 | 96 | 13 | GM_Racing | {dtor} | __dt__9GM_RacingFv |
| 0x002df4a0 | 32 | 13 | GM_Racing | OnBeforeCreate | OnBeforeCreate__9GM_RacingFv |
| 0x002df4c0 | 56 | GLOBAL | GM_ProfileSelect | OnKey | OnKey__16GM_ProfileSelectFUcUi |
| 0x002df500 | 1964 | GLOBAL | GM_ProfileSelect | OnCreate | OnCreate__16GM_ProfileSelectFv |
| 0x002dfcb0 | 2332 | GLOBAL | GM_ProfileSelect | OnCloseChild | OnCloseChild__16GM_ProfileSelectFP7GM_MenuUiUi |
| 0x002e05d0 | 3792 | GLOBAL | GM_ProfileSelect | OnTick | OnTick__16GM_ProfileSelectFUi |
| 0x002e14a0 | 544 | GLOBAL | GM_ProfileSelect | OnClick | OnClick__16GM_ProfileSelectFUi |
| 0x002e16c0 | 880 | GLOBAL | GM_ProfileCreate | OnCreate | OnCreate__16GM_ProfileCreateFv |
| 0x002e1a30 | 8 | GLOBAL | GM_ProfileCreate | OnClose | OnClose__16GM_ProfileCreateFv |
| 0x002e1a40 | 596 | GLOBAL | GM_ProfileCreate | OnClick | OnClick__16GM_ProfileCreateFUi |
| 0x002e1ca0 | 1192 | GLOBAL | GM_ProfileCreate | MartinovaKlavesa | MartinovaKlavesa__16GM_ProfileCreateFUiUi |
| 0x002e2150 | 56 | GLOBAL | GM_ProfileCreate | OnKey | OnKey__16GM_ProfileCreateFUcUi |
| 0x002e2190 | 84 | GLOBAL | GM_ProfileCreate | OnTick | OnTick__16GM_ProfileCreateFUi |
| 0x002e21f0 | 1168 | GLOBAL | GM_ProfileDelete | OnCreate | OnCreate__16GM_ProfileDeleteFv |
| 0x002e2680 | 164 | GLOBAL | GM_ProfileDelete | OnCloseChild | OnCloseChild__16GM_ProfileDeleteFP7GM_MenuUiUi |
| 0x002e2730 | 148 | GLOBAL | GM_ProfileDelete | OnClick | OnClick__16GM_ProfileDeleteFUi |
| 0x002e27d0 | 84 | GLOBAL | GM_ProfileDelete | OnTick | OnTick__16GM_ProfileDeleteFUi |
| 0x002e2830 | 96 | 13 | GM_ProfileDelete | {dtor} | __dt__16GM_ProfileDeleteFv |
| 0x002e2890 | 12 | 13 | GM_ProfileDelete | OnBeforeCreate | OnBeforeCreate__16GM_ProfileDeleteFv |
| 0x002e28a0 | 96 | 13 | GM_ProfileCreate | {dtor} | __dt__16GM_ProfileCreateFv |
| 0x002e2900 | 12 | 13 | GM_ProfileCreate | OnBeforeCreate | OnBeforeCreate__16GM_ProfileCreateFv |
| 0x002e2910 | 96 | 13 | GM_ProfileSelect | {dtor} | __dt__16GM_ProfileSelectFv |
| 0x002e2970 | 36 | 13 | GM_ProfileSelect | OnBeforeCreate | OnBeforeCreate__16GM_ProfileSelectFv |
| 0x002e29a0 | 668 | GLOBAL |  | LoadMaterialsFromBin__Fv | LoadMaterialsFromBin__Fv |
| 0x002e2c40 | 120 | GLOBAL |  | UnloadMaterials__Fv | UnloadMaterials__Fv |
| 0x002e2cc0 | 144 | GLOBAL |  | Line__FffffUi | Line__FffffUi |
| 0x002e2d50 | 228 | GLOBAL |  | Rectangle__FffffUi | Rectangle__FffffUi |
| 0x002e2e40 | 196 | GLOBAL |  | RectangleFilled__FffffUiP8ITexture | RectangleFilled__FffffUiP8ITexture |
| 0x002e2f10 | 200 | GLOBAL |  | RectangleProgress__FffffUifP8ITexture | RectangleProgress__FffffUifP8ITexture |
| 0x002e2fe0 | 1264 | GLOBAL |  | RectangleBB__FffffUi | RectangleBB__FffffUi |
| 0x002e34d0 | 572 | GLOBAL |  | Triangle__FffffUiUi | Triangle__FffffUiUi |
| 0x002e3710 | 300 | GLOBAL |  | SetViewport__Fiiii | SetViewport__Fiiii |
| 0x002e3840 | 640 | GLOBAL | GM_Menu | LoadResources | LoadResources__7GM_MenuFv |
| 0x002e3ac0 | 84 | GLOBAL | GM_Menu | UnloadResources | UnloadResources__7GM_MenuFv |
| 0x002e3b20 | 284 | GLOBAL | GM_Component | {ctor} | __ct__12GM_ComponentFP15GM_DesignStructP12GM_Component |
| 0x002e3c40 | 240 | GLOBAL | GM_Component | {ctor} | __ct__12GM_ComponentFP12GM_Component |
| 0x002e3d30 | 136 | GLOBAL | GM_Component | BaseDraw | BaseDraw__12GM_ComponentFUi |
| 0x002e3dc0 | 104 | GLOBAL | GM_Component | BaseAfterMove | BaseAfterMove__12GM_ComponentFUi |
| 0x002e3e30 | 264 | GLOBAL | GM_Component | InsertComponent | InsertComponent__12GM_ComponentFP12GM_Component |
| 0x002e3f40 | 416 | GLOBAL | GM_Component | InsertComponentAfter | InsertComponentAfter__12GM_ComponentFP12GM_ComponentP12GM_Component |
| 0x002e40e0 | 64 | GLOBAL | GM_Component | SetTextID | SetTextID__12GM_ComponentFUi |
| 0x002e4120 | 244 | GLOBAL | GM_Menu | PreCache | PreCache__7GM_MenuFv |
| 0x002e4220 | 64 | GLOBAL | GM_Menu | {ctor} | __ct__7GM_MenuFv |
| 0x002e4260 | 128 | GLOBAL | GM_Menu | {dtor} | __dt__7GM_MenuFv |
| 0x002e42e0 | 288 | GLOBAL | GM_Menu | Create | Create__7GM_MenuFv |
| 0x002e4400 | 1552 | GLOBAL | GM_Menu | CreateCompsByDef | CreateCompsByDef__7GM_MenuFPc |
| 0x002e4a10 | 128 | GLOBAL | GM_Menu | Destroy | Destroy__7GM_MenuFv |
| 0x002e4a90 | 336 | GLOBAL | GM_Menu | DeleteComponent | DeleteComponent__7GM_MenuFUi |
| 0x002e4be0 | 356 | GLOBAL | GM_Menu | Draw | Draw__7GM_MenuFUi |
| 0x002e4d50 | 788 | GLOBAL | GM_Menu | Tick | Tick__7GM_MenuFiiiUiUiUiUi |
| 0x002e5070 | 56 | GLOBAL | GM_Menu | MenuOpen | MenuOpen__7GM_MenuFP7GM_MenuUi |
| 0x002e50b0 | 52 | GLOBAL | GM_Menu | MenuOpenForMartin | MenuOpenForMartin__7GM_MenuFP7GM_MenuUi |
| 0x002e50f0 | 284 | GLOBAL | GM_Menu | MenuExchange | MenuExchange__7GM_MenuFv |
| 0x002e5210 | 120 | GLOBAL | GM_Menu | MenuClose | MenuClose__7GM_MenuFUi |
| 0x002e5290 | 148 | GLOBAL | GM_Menu | MenuCloseAll | MenuCloseAll__7GM_MenuFUiUi |
| 0x002e5330 | 148 | GLOBAL | GM_Menu | UtilCenterH | UtilCenterH__7GM_MenuFUi |
| 0x002e53d0 | 148 | GLOBAL | GM_Menu | UtilCenterV | UtilCenterV__7GM_MenuFUi |
| 0x002e5470 | 92 | GLOBAL | GM_Menu | SetDisabled | SetDisabled__7GM_MenuFUib |
| 0x002e54d0 | 64 | GLOBAL | GM_Menu | GetDisabled | GetDisabled__7GM_MenuFUi |
| 0x002e5510 | 92 | GLOBAL | GM_Menu | SetHidden | SetHidden__7GM_MenuFUib |
| 0x002e5570 | 92 | GLOBAL | GM_Menu | SetAlternateStyle | SetAlternateStyle__7GM_MenuFUib |
| 0x002e55d0 | 92 | GLOBAL | GM_Menu | SetAlternateColor | SetAlternateColor__7GM_MenuFUib |
| 0x002e5630 | 64 | GLOBAL | GM_Menu | SetTextID | SetTextID__7GM_MenuFUiUi |
| 0x002e5670 | 64 | GLOBAL | GM_Menu | GetTextID | GetTextID__7GM_MenuFUi |
| 0x002e56b0 | 64 | GLOBAL | GM_Menu | SetText | SetText__7GM_MenuFUiPUc |
| 0x002e56f0 | 64 | GLOBAL | GM_Menu | SetString | SetString__7GM_MenuFUiPUc |
| 0x002e5730 | 64 | GLOBAL | GM_Menu | GetText | GetText__7GM_MenuFUi |
| 0x002e5770 | 64 | GLOBAL | GM_Menu | SetFloat | SetFloat__7GM_MenuFUif |
| 0x002e57b0 | 64 | GLOBAL | GM_Menu | GetFloat | GetFloat__7GM_MenuFUi |
| 0x002e57f0 | 64 | GLOBAL | GM_Menu | SetDword | SetDword__7GM_MenuFUiUi |
| 0x002e5830 | 64 | GLOBAL | GM_Menu | GetDword | GetDword__7GM_MenuFUi |
| 0x002e5870 | 64 | GLOBAL | GM_Menu | SetBool | SetBool__7GM_MenuFUib |
| 0x002e58b0 | 64 | GLOBAL | GM_Menu | GetBool | GetBool__7GM_MenuFUi |
| 0x002e58f0 | 64 | GLOBAL | GM_Menu | SetPointer | SetPointer__7GM_MenuFUiPv |
| 0x002e5930 | 80 | GLOBAL | GM_Menu | GetRangeFloat | GetRangeFloat__7GM_MenuFUiRfRf |
| 0x002e5980 | 80 | GLOBAL | GM_Menu | SetRangeDword | SetRangeDword__7GM_MenuFUiUiUi |
| 0x002e59d0 | 140 | GLOBAL | GM_Menu | ActivateByID | ActivateByID__7GM_MenuFUi |
| 0x002e5a60 | 208 | GLOBAL | GM_Menu | ActivateByIDOrNext | ActivateByIDOrNext__7GM_MenuFUi |
| 0x002e5b30 | 208 | GLOBAL | GM_Menu | ActivateByIDOrPrev | ActivateByIDOrPrev__7GM_MenuFUi |
| 0x002e5c00 | 60 | GLOBAL | GM_Menu | FindComponentByID | FindComponentByID__7GM_MenuFUi |
| 0x002e5c40 | 236 | GLOBAL | GM_Menu | FindComponentByIDPrev | FindComponentByIDPrev__7GM_MenuFUi |
| 0x002e5d30 | 236 | GLOBAL | GM_Menu | FindComponentByIDNext | FindComponentByIDNext__7GM_MenuFUi |
| 0x002e5e20 | 664 | GLOBAL | GM_Menu | PridejComponent | PridejComponent__7GM_MenuFfUiUiUiUiUiUi |
| 0x002e60c0 | 644 | GLOBAL | GM_Menu | PridejComponentPod | PridejComponentPod__7GM_MenuFUiUiUiUiUiUi |
| 0x002e6350 | 1488 | GLOBAL | GM_Menu | ExecuteMenu | ExecuteMenu__7GM_MenuFP7GM_Menubbb |
| 0x002e6920 | 1388 | GLOBAL | GM_Menu | SetCameraTargetID | SetCameraTargetID__7GM_MenuFUi |
| 0x002e6e90 | 1352 | GLOBAL | GM_Menu | TickCamera | TickCamera__7GM_MenuFUi |
| 0x002e73e0 | 216 | GLOBAL | C_ShotSkeleton | Init | Init__14C_ShotSkeletonFP9I3D_frame |
| 0x002e74c0 | 1620 | GLOBAL | C_ShotSkeleton | TestCollision | TestCollision__14C_ShotSkeletonFRC8S_vectorRC8S_vectorf |
| 0x002e7b20 | 12 | GLOBAL | G_TextDatabase | {ctor} | __ct__14G_TextDatabaseFv |
| 0x002e7b30 | 96 | GLOBAL | G_TextDatabase | {dtor} | __dt__14G_TextDatabaseFv |
| 0x002e7b90 | 280 | GLOBAL | G_TextDatabase | LoadDatabase | LoadDatabase__14G_TextDatabaseFPcb |
| 0x002e7cb0 | 60 | GLOBAL | G_TextDatabase | Unload | Unload__14G_TextDatabaseFv |
| 0x002e7cf0 | 172 | GLOBAL | G_TextDatabase | Transpose | Transpose__14G_TextDatabaseFUiUiUi |
| 0x002e7da0 | 224 | GLOBAL | G_TextDatabase | GetText | GetText__14G_TextDatabaseFUi |
| 0x002e7e80 | 216 | GLOBAL | G_TextDatabase | GetTextOrNULL | GetTextOrNULL__14G_TextDatabaseFUi |
| 0x002e7f60 | 1160 | GLOBAL |  | PreloadSound__FPc | PreloadSound__FPc |
| 0x002e83f0 | 56 | GLOBAL |  | ZpicenyPleaseWait__Fb | ZpicenyPleaseWait__Fb |
| 0x002e8430 | 64 | GLOBAL |  | PreloadCrashModelsClear__Fv | PreloadCrashModelsClear__Fv |
| 0x002e8470 | 244 | GLOBAL |  | PreloadCrashModelsCheck__FPCc | PreloadCrashModelsCheck__FPCc |
| 0x002e8570 | 220 | GLOBAL |  | PreloadCrashModelsPreload__Fv | PreloadCrashModelsPreload__Fv |
| 0x002e8650 | 1188 | 13 | C_I3D_cache<9I3D_model> | Open | Open__23C_I3D_cache<9I3D_model>FP9I3D_modelPCcUiPFPCcPv_vPvPv |
| 0x002e8b00 | 4440 | GLOBAL |  | PreloadPicovinNaZasranyPSKram__Fv | PreloadPicovinNaZasranyPSKram__Fv |
| 0x002e9c60 | 340 | 13 | S_hlp$2662game_cpp | cbCrashModel | cbCrashModel__18S_hlp$2662game_cppFP9I3D_frame |
| 0x002e9dc0 | 580 | GLOBAL |  | PreloadPicovinNaZasranyPSKram2__Fv | PreloadPicovinNaZasranyPSKram2__Fv |
| 0x002ea010 | 68 | GLOBAL | C_game | SetHuman | SetHuman__6C_gameFP7C_actor |
| 0x002ea060 | 56 | LOCAL |  | _show_progress___Ff | _show_progress___Ff |
| 0x002ea0a0 | 14016 | GLOBAL | C_game | Init | Init__6C_gameFbb |
| 0x002ed760 | 268 | 13 | gif::vector<Q26C_game18S_fake_shadow_buff,8> | reserve | reserve__Q23gif39vector<Q26C_game18S_fake_shadow_buff,8>Fi |
| 0x002ed870 | 356 | 13 | C_game::S_hlp$3427game_cpp | cbEnumSectors | cbEnumSectors__Q26C_game18S_hlp$3427game_cppFP9I3D_frame |
| 0x002ed9e0 | 108 | 13 | gif::vector<P7C_actor,8> | clear | clear__Q23gif19vector<P7C_actor,8>Fv |
| 0x002eda50 | 5812 | GLOBAL | C_game | Done | Done__6C_gameFv |
| 0x002ef110 | 1368 | GLOBAL | C_game | DoneIntern | DoneIntern__6C_gameFv |
| 0x002ef670 | 8 | 13 | std::__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>> | alloc | alloc__Q23std122__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>>Fv |
| 0x002ef680 | 484 | GLOBAL | C_game | ChangeActState | ChangeActState__6C_gameFv |
| 0x002ef870 | 6856 | GLOBAL | C_game | Tick | Tick__6C_gameFUi |
| 0x002f1340 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std35allocator<Q26C_game13S_i_show_fade>,Ui,1> | first | first__Q310Metrowerks7details69compressed_pair_imp<Q23std35allocator<Q26C_game13S_i_show_fade>,Ui,1>Fv |
| 0x002f1350 | 2504 | GLOBAL | C_game | DoubleKiss | DoubleKiss__6C_gameFUi |
| 0x002f1d20 | 272 | GLOBAL | C_game | AddTemporaryActor | AddTemporaryActor__6C_gameFP7C_actor |
| 0x002f1e30 | 148 | GLOBAL | C_game | RemoveTemporaryActor | RemoveTemporaryActor__6C_gameFP7C_actor |
| 0x002f1ed0 | 252 | GLOBAL | C_game | InternTempActAdd | InternTempActAdd__6C_gameFP7C_actor |
| 0x002f1fd0 | 304 | GLOBAL | C_game | InternTempActDel | InternTempActDel__6C_gameFP7C_actor |
| 0x002f2100 | 188 | GLOBAL | C_game | _RemoveTemporaryActor | _RemoveTemporaryActor__6C_gameFP7C_actor |
| 0x002f21c0 | 192 | GLOBAL | C_game | VyhjebejTemporaryho | VyhjebejTemporaryho__6C_gameFP7C_actor |
| 0x002f2280 | 188 | GLOBAL | C_game | ClearTemporaryActors | ClearTemporaryActors__6C_gameFv |
| 0x002f2340 | 132 | GLOBAL | C_game | UpdateControls | UpdateControls__6C_gameFv |
| 0x002f23d0 | 644 | GLOBAL | C_game | SetActiveTrafficP | SetActiveTrafficP__6C_gameFUi |
| 0x002f2660 | 64 | GLOBAL | C_game | SetTrafficVisible | SetTrafficVisible__6C_gameFb |
| 0x002f26a0 | 3132 | GLOBAL | C_game | NewShoot | NewShoot__6C_gameFP7C_actorRC8S_vectorRC8S_vectorfiP9I3D_framei |
| 0x002f32e0 | 112 | 13 | S_shoot | {ctor} | __ct__7S_shootFRC7S_shoot |
| 0x002f3350 | 348 | 13 | gif::vector<7S_shoot,8> | reserve | reserve__Q23gif18vector<7S_shoot,8>Fi |
| 0x002f34b0 | 7544 | GLOBAL | C_game | TickShoot | TickShoot__6C_gameFUi |
| 0x002f5230 | 840 | LOCAL |  | _explosion__FP7C_actorfP7C_actorR8S_vectorff | _explosion__FP7C_actorfP7C_actorR8S_vectorff |
| 0x002f5580 | 2752 | GLOBAL | C_game | NewExplosion | NewExplosion__6C_gameFP7C_actorR8S_vectorffbbbi |
| 0x002f6040 | 820 | GLOBAL | C_game | NewSmokeTrail | NewSmokeTrail__6C_gameFR8S_vectorff |
| 0x002f6380 | 728 | GLOBAL | C_game | NewSmallFires | NewSmallFires__6C_gameFR8S_vectori |
| 0x002f6660 | 500 | GLOBAL | C_game | NewFire | NewFire__6C_gameFR8S_vectorP9I3D_frameiffb |
| 0x002f6860 | 8 | GLOBAL | C_game | NewCartidge | NewCartidge__6C_gameFRP9I3D_frameR8S_vectorbb |
| 0x002f6870 | 1960 | GLOBAL | C_game | NewGrenade | NewGrenade__6C_gameFiR8S_vectorR8S_vectorbP9I3D_frame |
| 0x002f7020 | 1780 | GLOBAL | C_game | Play3DSound | Play3DSound__6C_gameFP9I3D_framePCc13I3D_SOUNDTYPERC8S_vectorfffbfb |
| 0x002f7720 | 164 | GLOBAL | C_game | Stop3DSound | Stop3DSound__6C_gameFibf |
| 0x002f77d0 | 140 | GLOBAL | C_game | Play3DSound | Play3DSound__6C_gameFP9I3D_sound |
| 0x002f7860 | 852 | 13 | C_game | TickEffects | TickEffects__6C_gameFUi |
| 0x002f7bc0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std34allocator<Q26C_game12S_sound_play>,Ui,1> | first | first__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q26C_game12S_sound_play>,Ui,1>Fv |
| 0x002f7bd0 | 192 | GLOBAL | C_game | ClearEffects | ClearEffects__6C_gameFv |
| 0x002f7c90 | 964 | GLOBAL | C_game | AddFakeShadow | AddFakeShadow__6C_gameFP9I3D_frameP7C_actor |
| 0x002f8060 | 348 | 13 | gif::vector<Q26C_game17S_fake_shadow_hum,8> | reserve | reserve__Q23gif38vector<Q26C_game17S_fake_shadow_hum,8>Fi |
| 0x002f81c0 | 360 | GLOBAL | C_game | DelFakeShadow | DelFakeShadow__6C_gameFP9I3D_frame |
| 0x002f8330 | 1864 | GLOBAL | C_game | TickFakeShadows | TickFakeShadows__6C_gameFUi |
| 0x002f8a80 | 520 | 13 | C_game::S_hlp$11688game_cpp | cbFindMinMax | cbFindMinMax__Q26C_game19S_hlp$11688game_cppFP9I3D_frameR8S_matrixRfRfRfRf |
| 0x002f8c90 | 8 | 13 | C_game | TickShadows | TickShadows__6C_gameFUi |
| 0x002f8ca0 | 540 | GLOBAL | C_game | AddFrameForShadow | AddFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type13E_shadow_typeP9I3D_framePc |
| 0x002f8ec0 | 8 | GLOBAL | C_game | SetFrameForShadow | SetFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type13E_shadow_type |
| 0x002f8ed0 | 8 | GLOBAL | C_game | DelFrameForShadow | DelFrameForShadow__6C_gameFP9I3D_frame13E_shadow_type |
| 0x002f8ee0 | 468 | GLOBAL |  | cbSemaphores__FP9I3D_frameUi | cbSemaphores__FP9I3D_frameUi |
| 0x002f90c0 | 84 | GLOBAL | C_game | LoadSemaphores | LoadSemaphores__6C_gameFv |
| 0x002f9120 | 216 | GLOBAL | C_game | FreeSemaphores | FreeSemaphores__6C_gameFv |
| 0x002f9200 | 304 | GLOBAL | C_game | TickSemaphores | TickSemaphores__6C_gameFv |
| 0x002f9330 | 52 | GLOBAL | C_game | LoadCityMusicAreas | LoadCityMusicAreas__6C_gameFv |
| 0x002f9370 | 180 | 13 | C_game::S_hlp$12237game_cpp | cbCityMusic | cbCityMusic__Q26C_game19S_hlp$12237game_cppFP9I3D_frameUi |
| 0x002f9430 | 2144 | 13 | C_game | TickCityMusic | TickCityMusic__6C_gameFUi |
| 0x002f9c90 | 96 | GLOBAL | C_game | CityMusicOff | CityMusicOff__6C_gameFv |
| 0x002f9cf0 | 36 | GLOBAL | C_game | CityMusicPaused | CityMusicPaused__6C_gameFb |
| 0x002f9d20 | 8 | GLOBAL | C_game | SetFilmMusic | SetFilmMusic__6C_gameFi |
| 0x002f9d30 | 8 | GLOBAL | C_game | GetFilmMusic | GetFilmMusic__6C_gameFi |
| 0x002f9d40 | 8 | GLOBAL | C_game | ClearFrameFromFireHole | ClearFrameFromFireHole__6C_gameFP9I3D_frame |
| 0x002f9d50 | 1420 | GLOBAL | C_game | ClearDiffData | ClearDiffData__6C_gameFv |
| 0x002fa2e0 | 568 | 13 | C_game::S_hlp$13685game_cpp | cbFreeFramePlacement | cbFreeFramePlacement__Q26C_game19S_hlp$13685game_cppFP9I3D_frameRQ23gif38vector<Q26C_game17S_frame_placement,8> |
| 0x002fa520 | 152 | GLOBAL | C_game | GlobalProgramRun | GlobalProgramRun__6C_gameFPc |
| 0x002fa5c0 | 168 | GLOBAL | C_game | InvalidateColl | InvalidateColl__6C_gameFPUi |
| 0x002fa670 | 1428 | GLOBAL | C_game | InvalidateActor | InvalidateActor__6C_gameFP7C_actorb |
| 0x002fac10 | 260 | GLOBAL | C_game | InvalidateFrame | InvalidateFrame__6C_gameFP9I3D_frame |
| 0x002fad20 | 496 | 13 | C_game::S_hlp$14436game_cpp | cbDelFrame | cbDelFrame__Q26C_game19S_hlp$14436game_cppFP9I3D_frameRQ23gif38vector<Q26C_game17S_frame_placement,8> |
| 0x002faf10 | 600 | GLOBAL | C_game | AddFramePlacement | AddFramePlacement__6C_gameFP9I3D_frame |
| 0x002fb170 | 300 | 13 | gif::vector<Q26C_game17S_frame_placement,8> | reserve | reserve__Q23gif38vector<Q26C_game17S_frame_placement,8>Fi |
| 0x002fb2a0 | 176 | GLOBAL | C_game | DelFramePlacement | DelFramePlacement__6C_gameFP9I3D_frame |
| 0x002fb350 | 92 | GLOBAL | C_game | IsFramePlacement | IsFramePlacement__6C_gameFP9I3D_frame |
| 0x002fb3b0 | 44 | GLOBAL | C_game | Timer_TimeOut | Timer_TimeOut__6C_gameFv |
| 0x002fb3e0 | 44 | GLOBAL | C_game | PFire_Fire | PFire_Fire__6C_gameFv |
| 0x002fb410 | 44 | GLOBAL | C_game | PHorn_Horn | PHorn_Horn__6C_gameFv |
| 0x002fb440 | 44 | GLOBAL | C_game | WFall_Player | WFall_Player__6C_gameFv |
| 0x002fb470 | 228 | GLOBAL | C_game | AddNPCKillEvent | AddNPCKillEvent__6C_gameFP9C_programiPc |
| 0x002fb560 | 188 | GLOBAL | C_game | Do_NPCKillEvent | Do_NPCKillEvent__6C_gameFPc |
| 0x002fb620 | 1600 | GLOBAL | C_game | AddDriverMan | AddDriverMan__6C_gameFP9I3D_modelP5C_carib |
| 0x002fbc60 | 224 | GLOBAL | C_game | DelDriverMan | DelDriverMan__6C_gameFP9I3D_model |
| 0x002fbd40 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std34allocator<Q26C_game12S_driver_man>,Ui,1> | first | first__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q26C_game12S_driver_man>,Ui,1>Fv |
| 0x002fbd50 | 368 | 13 | C_game | TickDriverMan | TickDriverMan__6C_gameFv |
| 0x002fbec0 | 8 | GLOBAL | C_game | CreateTrackEffect | CreateTrackEffect__6C_gameFR8S_vectorR8S_vectorfR8S_vectorfibfbbP9I3D_frame |
| 0x002fbed0 | 8 | GLOBAL | C_game | CreateTrackEffect | CreateTrackEffect__6C_gameFR8S_vectorR8S_vectorR8S_vectorR8S_vectorfibfbbP9I3D_frame |
| 0x002fbee0 | 524 | GLOBAL | C_game | CreateTrackEffect_Grenade | CreateTrackEffect_Grenade__6C_gameFR8S_vector |
| 0x002fc0f0 | 132 | GLOBAL | C_game | ClearProjectorsForFrame | ClearProjectorsForFrame__6C_gameFP9I3D_frame |
| 0x002fc180 | 220 | GLOBAL | C_game | InventoryStackClear | InventoryStackClear__6C_gameFv |
| 0x002fc260 | 296 | GLOBAL | C_game | InventoryStackPush | InventoryStackPush__6C_gameFP7C_human |
| 0x002fc390 | 668 | 13 | gif::vector<10S_GameItem,8> | operator= | __as__Q23gif22vector<10S_GameItem,8>FRCQ23gif22vector<10S_GameItem,8> |
| 0x002fc630 | 432 | GLOBAL | C_game | InventoryStackPop | InventoryStackPop__6C_gameFP7C_human |
| 0x002fc7e0 | 540 | 13 | gif::vector<Ui,8> | operator= | __as__Q23gif12vector<Ui,8>FRCQ23gif12vector<Ui,8> |
| 0x002fca00 | 8 | GLOBAL | C_game | FloatStackClear | FloatStackClear__6C_gameFv |
| 0x002fca10 | 108 | GLOBAL | C_game | FloatStackPush | FloatStackPush__6C_gameFf |
| 0x002fca80 | 48 | GLOBAL | C_game | FloatStackPop | FloatStackPop__6C_gameFPf |
| 0x002fcab0 | 8 | GLOBAL | C_game | CharacterStackClear | CharacterStackClear__6C_gameFv |
| 0x002fcac0 | 724 | GLOBAL | C_game | CharacterStackPush | CharacterStackPush__6C_gameFP7C_human |
| 0x002fcda0 | 668 | GLOBAL | C_game | CharacterStackPop | CharacterStackPop__6C_gameFP7C_human |
| 0x002fd040 | 1012 | GLOBAL | C_game | BoxPlacementAdd | BoxPlacementAdd__6C_gameFP9I3D_framei |
| 0x002fd440 | 240 | GLOBAL | C_game | BoxPlacementDel | BoxPlacementDel__6C_gameFP9I3D_frame |
| 0x002fd530 | 848 | GLOBAL | C_game | BoxPlacementRecompile | BoxPlacementRecompile__6C_gameFv |
| 0x002fd880 | 180 | GLOBAL | C_game | BoxPlacementDestAdd | BoxPlacementDestAdd__6C_gameFP9I3D_framei |
| 0x002fd940 | 448 | GLOBAL | C_game | BoxPlacementDestSetOU | BoxPlacementDestSetOU__6C_gameFbP9I3D_frame |
| 0x002fdb00 | 300 | GLOBAL | C_game | BoxPlacementReposeDestPlayer | BoxPlacementReposeDestPlayer__6C_gameFv |
| 0x002fdc30 | 612 | GLOBAL | C_game | BoxPlacement_SaveInfo | BoxPlacement_SaveInfo__6C_gameFP9I3D_frame |
| 0x002fdea0 | 356 | 13 | gif::vector<Q26C_game20S_box_placement_temp,8> | reserve | reserve__Q23gif41vector<Q26C_game20S_box_placement_temp,8>Fi |
| 0x002fe010 | 176 | GLOBAL | C_game | BoxPlacement_DelInfo | BoxPlacement_DelInfo__6C_gameFP9I3D_frame |
| 0x002fe0c0 | 368 | GLOBAL | C_game | BoxPlacement_RestoreInfo | BoxPlacement_RestoreInfo__6C_gameFP9I3D_frame |
| 0x002fe230 | 224 | GLOBAL | C_game | BoxPlacement_GetStartPoint | BoxPlacement_GetStartPoint__6C_gameFP9I3D_frameR8S_vector |
| 0x002fe310 | 740 | GLOBAL | C_game | BoxPlacementGetNextDest | BoxPlacementGetNextDest__6C_gameFP9I3D_frameP9I3D_frame |
| 0x002fe600 | 436 | GLOBAL | C_game | BoxPlacementGetEnemyDest | BoxPlacementGetEnemyDest__6C_gameFP7C_humani |
| 0x002fe7c0 | 80 | GLOBAL | C_game | BoxPlacement_GetBox | BoxPlacement_GetBox__6C_gameFi |
| 0x002fe810 | 80 | GLOBAL | C_game | BoxPlacement_GetBoxDest | BoxPlacement_GetBoxDest__6C_gameFi |
| 0x002fe860 | 324 | GLOBAL | C_game | BoxPlacement_MoveBoxToDest | BoxPlacement_MoveBoxToDest__6C_gameFi |
| 0x002fe9b0 | 672 | GLOBAL | C_game | BoxRecompileBoxForCar | BoxRecompileBoxForCar__6C_gameFP5C_carb |
| 0x002fec50 | 144 | GLOBAL | C_game | BoxPlacementPoPiciNaZkurvenemAute | BoxPlacementPoPiciNaZkurvenemAute__6C_gameFP5C_cari |
| 0x002fece0 | 100 | GLOBAL | C_game | InterCityVyjeb | InterCityVyjeb__6C_gameFv |
| 0x002fed50 | 100 | GLOBAL | C_game | InterCityNajeb | InterCityNajeb__6C_gameFv |
| 0x002fedc0 | 136 | GLOBAL | C_game | DeathActorAdd | DeathActorAdd__6C_gameFP7C_actor |
| 0x002fee50 | 152 | GLOBAL | C_game | DeathActorProcess | DeathActorProcess__6C_gameFv |
| 0x002feef0 | 132 | GLOBAL | C_game | GetActorFromGUID | GetActorFromGUID__6C_gameFUi |
| 0x002fef80 | 156 | GLOBAL | C_game | SaveGameGetSize | SaveGameGetSize__6C_gameFb |
| 0x002ff020 | 460 | GLOBAL | C_game | SaveGameSave | SaveGameSave__6C_gameFPPvb |
| 0x002ff1f0 | 708 | GLOBAL | C_game | SaveGameLoad | SaveGameLoad__6C_gameFPPv |
| 0x002ff4c0 | 68 | GLOBAL | C_game | SaveGameLoadBefore | SaveGameLoadBefore__6C_gameFPPv |
| 0x002ff510 | 8 | GLOBAL | C_game | LightnessAdd | LightnessAdd__6C_gameFP7C_actorP9I3D_frameUsR8S_vector |
| 0x002ff520 | 160 | GLOBAL | C_game | LightnessDel | LightnessDel__6C_gameFP9I3D_frame |
| 0x002ff5c0 | 96 | GLOBAL | C_game | LightnessClear | LightnessClear__6C_gameFv |
| 0x002ff620 | 8 | GLOBAL | C_game | LightnessTick | LightnessTick__6C_gameFUi |
| 0x002ff630 | 8 | GLOBAL | C_game | AddShootLight | AddShootLight__6C_gameFR8S_vectorfiUs |
| 0x002ff640 | 388 | 13 | C_game | ShootLightTick | ShootLightTick__6C_gameFUi |
| 0x002ff7d0 | 236 | GLOBAL | C_game | UseLightCacheSystem | UseLightCacheSystem__6C_gameFv |
| 0x002ff8c0 | 388 | 13 | C_game::S_hlp$18015game_cpp | cbLight | cbLight__Q26C_game19S_hlp$18015game_cppFP9I3D_frameRQ23std50vector<P9I3D_light,Q23std22allocator<P9I3D_light>> |
| 0x002ffa50 | 56 | GLOBAL | C_game | RecomputeLightCache | RecomputeLightCache__6C_gameFv |
| 0x002ffa90 | 388 | 13 | C_game | LightCacheTick | LightCacheTick__6C_gameFUi |
| 0x002ffc20 | 248 | GLOBAL | C_game | CreateIShow | CreateIShow__6C_gameFUi |
| 0x002ffd20 | 524 | GLOBAL | C_game | OpenIShow | OpenIShow__6C_gameFPCci |
| 0x002fff30 | 280 | GLOBAL | C_game | DestroyIShow | DestroyIShow__6C_gameFi |
| 0x00300050 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std29allocator<Q26C_game8S_i_show>,Ui,1> | first | first__Q310Metrowerks7details63compressed_pair_imp<Q23std29allocator<Q26C_game8S_i_show>,Ui,1>Fv |
| 0x00300060 | 192 | GLOBAL | C_game | DestroyIShowAll | DestroyIShowAll__6C_gameFv |
| 0x00300120 | 84 | GLOBAL | C_game | GetIShow | GetIShow__6C_gameFi |
| 0x00300180 | 80 | GLOBAL | C_game | IShowStoreVolume | IShowStoreVolume__6C_gameFif |
| 0x003001d0 | 200 | GLOBAL | C_game | IShowLoop | IShowLoop__6C_gameFib |
| 0x003002a0 | 116 | GLOBAL | C_game | IShowSetVolume | IShowSetVolume__6C_gameFv |
| 0x00300320 | 244 | GLOBAL | C_game | IShowFadeDownAll | IShowFadeDownAll__6C_gameFi |
| 0x00300420 | 304 | GLOBAL | C_game | IShowFade | IShowFade__6C_gameFiffi |
| 0x00300550 | 92 | GLOBAL | C_game | LoadRoadBlocks | LoadRoadBlocks__6C_gameFv |
| 0x003005b0 | 196 | 13 | C_game::S_hlp$18574game_cpp | cbRoadBlock | cbRoadBlock__Q26C_game19S_hlp$18574game_cppFP9I3D_frameUi |
| 0x00300680 | 644 | GLOBAL | C_game | LoadTaxiPoints | LoadTaxiPoints__6C_gameFv |
| 0x00300910 | 444 | 13 | C_game::S_hlp$18659game_cpp | cbFindTaxiPoints | cbFindTaxiPoints__Q26C_game19S_hlp$18659game_cppFP9I3D_frameRQ23std50vector<P9I3D_frame,Q23std22allocator<P9I3D_frame>> |
| 0x00300ad0 | 88 | GLOBAL | C_game | IsValidTaxiPassenger | IsValidTaxiPassenger__6C_gameFv |
| 0x00300b30 | 164 | GLOBAL | C_game | DelSemaphore | DelSemaphore__6C_gameFP9I3D_frame |
| 0x00300be0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std24allocator<11S_semaphore>,Ui,1> | first | first__Q310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_semaphore>,Ui,1>Fv |
| 0x00300bf0 | 592 | GLOBAL | C_game | LoadTelephones | LoadTelephones__6C_gameFv |
| 0x00300e40 | 276 | 13 | C_game::S_hlp$18990game_cpp | cbTelephones | cbTelephones__Q26C_game19S_hlp$18990game_cppFP9I3D_frameUi |
| 0x00300f60 | 148 | GLOBAL | C_game | DelTelephones | DelTelephones__6C_gameFP9I3D_frame |
| 0x00301000 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std33allocator<Q26C_game11S_Telephone>,Ui,1> | first | first__Q310Metrowerks7details67compressed_pair_imp<Q23std33allocator<Q26C_game11S_Telephone>,Ui,1>Fv |
| 0x00301010 | 100 | GLOBAL | C_game | IsTelephoneExist | IsTelephoneExist__6C_gameFP9I3D_frame |
| 0x00301080 | 60 | GLOBAL | C_game | AddHumanTalk | AddHumanTalk__6C_gameFP7C_humani |
| 0x003010c0 | 168 | GLOBAL | C_game | DelHumanTalk | DelHumanTalk__6C_gameFP7C_humani |
| 0x00301170 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std34allocator<Q26C_game12S_enemy_talk>,Ui,1> | first | first__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q26C_game12S_enemy_talk>,Ui,1>Fv |
| 0x00301180 | 196 | GLOBAL | C_game | DeathHumanTalk | DeathHumanTalk__6C_gameFP7C_human |
| 0x00301250 | 868 | GLOBAL | C_game | CreateSignalMarkIDX | CreateSignalMarkIDX__6C_gameFv |
| 0x003015c0 | 904 | GLOBAL | C_game | DestroyControlSignal | DestroyControlSignal__6C_gameFPUiR8S_vectorR8S_vectorR8S_vectorP7C_human |
| 0x00301950 | 260 | GLOBAL | C_game | EntityBlock | EntityBlock__6C_gameFbb |
| 0x00301a60 | 116 | GLOBAL | C_game | AddInvisibleActor | AddInvisibleActor__6C_gameFP7C_actor |
| 0x00301ae0 | 124 | GLOBAL | C_game | FindActorsVect | FindActorsVect__6C_gameFv |
| 0x00301b60 | 152 | GLOBAL | C_game | StopAllSounds | StopAllSounds__6C_gameFv |
| 0x00301c00 | 1068 | GLOBAL | C_game | PauseAllSounds | PauseAllSounds__6C_gameFv |
| 0x00302030 | 396 | 13 | gif::vector<Q26C_game13S_stop_sounds,8> | reserve | reserve__Q23gif34vector<Q26C_game13S_stop_sounds,8>Fi |
| 0x003021c0 | 808 | GLOBAL | C_game | PlayAllSounds | PlayAllSounds__6C_gameFb |
| 0x003024f0 | 8 | 13 | C_string::C_str_base | GetData | GetData__Q28C_string10C_str_baseFv |
| 0x00302500 | 1488 | GLOBAL | C_game | ScratchAdd | ScratchAdd__6C_gameFP5C_carP9I3D_frameUiRC8S_vectorRC8S_vector |
| 0x00302ad0 | 8 | 13 | I3D_projector | MakeProjection | MakeProjection__13I3D_projectorFUiP9I3D_frame |
| 0x00302ae0 | 256 | GLOBAL | C_game | ScratchRecompile | ScratchRecompile__6C_gameFP5C_carP9I3D_frame |
| 0x00302be0 | 176 | GLOBAL | C_game | ScratchDeleteFromCar | ScratchDeleteFromCar__6C_gameFP5C_car |
| 0x00302c90 | 848 | GLOBAL | C_game | ScratchGetFreeIdx | ScratchGetFreeIdx__6C_gameFP5C_carP9I3D_frameRC8S_vector |
| 0x00302fe0 | 156 | GLOBAL | C_game | AddCreatedFrameToVector | AddCreatedFrameToVector__6C_gameFP9I3D_frame |
| 0x00303080 | 116 | GLOBAL | C_game | DelCreatedFrameToVector | DelCreatedFrameToVector__6C_gameFP9I3D_frame |
| 0x00303100 | 84 | GLOBAL | C_game | LoadScreamAreas | LoadScreamAreas__6C_gameFv |
| 0x00303160 | 304 | 13 | C_game::S_hlp$20556game_cpp | cbScreamArea | cbScreamArea__Q26C_game19S_hlp$20556game_cppFP9I3D_frameUi |
| 0x00303290 | 520 | GLOBAL | C_game | TickScreamAreas | TickScreamAreas__6C_gameFv |
| 0x003034a0 | 436 | GLOBAL | C_game | OsefujPocasi | OsefujPocasi__6C_gameFbUi |
| 0x00303660 | 136 | GLOBAL | C_game | AddDeadHumanForTick | AddDeadHumanForTick__6C_gameFP7C_human |
| 0x003036f0 | 88 | GLOBAL | C_game | DelDeadHumanForTick | DelDeadHumanForTick__6C_gameFP7C_human |
| 0x00303750 | 760 | GLOBAL | C_game | TickDeadHumanForTick | TickDeadHumanForTick__6C_gameFUi |
| 0x00303a50 | 820 | GLOBAL | C_game | AddHydrant | AddHydrant__6C_gameFP9I3D_frame |
| 0x00303d90 | 284 | 13 | gif::vector<Q26C_game9S_hydrant,8> | reserve | reserve__Q23gif29vector<Q26C_game9S_hydrant,8>Fi |
| 0x00303eb0 | 864 | 13 | C_game | TickHydrant | TickHydrant__6C_gameFUi |
| 0x00304210 | 276 | GLOBAL | C_game | DelAllHydrants | DelAllHydrants__6C_gameFv |
| 0x00304330 | 80 | GLOBAL | C_game | ScoreSetOn | ScoreSetOn__6C_gameFb |
| 0x00304380 | 52 | GLOBAL | C_game | ScoreSet | ScoreSet__6C_gameFi |
| 0x003043c0 | 132 | GLOBAL | C_game | SetCameraRotRepair | SetCameraRotRepair__6C_gameFv |
| 0x00304450 | 192 | GLOBAL | C_game | ClearCartridges | ClearCartridges__6C_gameFv |
| 0x00304510 | 64 | GLOBAL |  | compare_car_snd__FPCvPCv | compare_car_snd__FPCvPCv |
| 0x00304550 | 708 | GLOBAL | C_game | TickCarSoundManager | TickCarSoundManager__6C_gameFv |
| 0x00304820 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std21allocator<9S_car_snd>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details55compressed_pair_imp<Q23std21allocator<9S_car_snd>,Ui,1>FUi |
| 0x00304830 | 212 | GLOBAL | C_game | ItemBlurDone | ItemBlurDone__6C_gameFv |
| 0x00304910 | 2532 | GLOBAL | C_game | ItemBlurTick | ItemBlurTick__6C_gameFUi |
| 0x00305300 | 352 | GLOBAL | C_game | ItemBlurDelItem | ItemBlurDelItem__6C_gameFib |
| 0x00305460 | 508 | GLOBAL | C_game | AddDeathHumanFade | AddDeathHumanFade__6C_gameFP10I3D_objectP9I3D_frameb |
| 0x00305660 | 364 | 13 | gif::vector<Q26C_game18S_death_human_fade,8> | reserve | reserve__Q23gif39vector<Q26C_game18S_death_human_fade,8>Fi |
| 0x003057d0 | 412 | GLOBAL | C_game | ClearDeathHumanFade | ClearDeathHumanFade__6C_gameFv |
| 0x00305970 | 568 | GLOBAL | C_game | TickDeathHumanFade | TickDeathHumanFade__6C_gameFUi |
| 0x00305bb0 | 868 | GLOBAL | C_game | AddFireFlame | AddFireFlame__6C_gameFRC8S_vectorRC8S_vectorP9I3D_frameP7C_human |
| 0x00305f20 | 164 | GLOBAL | C_game | TickFireFlames | TickFireFlames__6C_gameFUi |
| 0x00305fd0 | 508 | GLOBAL | C_game | AddInterCityArea | AddInterCityArea__6C_gameFP8I3D_areaPc |
| 0x003061d0 | 396 | 13 | gif::vector<Q26C_game12S_inter_city,8> | reserve | reserve__Q23gif33vector<Q26C_game12S_inter_city,8>Fi |
| 0x00306360 | 108 | GLOBAL | C_game | ClearInterCityAreas | ClearInterCityAreas__6C_gameFv |
| 0x003063d0 | 320 | GLOBAL | C_game | TickInterCity | TickInterCity__6C_gameFUiPc |
| 0x00306510 | 604 | GLOBAL | C_game | PreloadAnims | PreloadAnims__6C_gameFv |
| 0x00306770 | 752 | 13 | C_I3D_cache<17I3D_animation_set> | Create | Create__32C_I3D_cache<17I3D_animation_set>FPCcPP17I3D_animation_setPFPCcPv_vPv |
| 0x00306a60 | 248 | GLOBAL | C_game | AddNewActorToActive | AddNewActorToActive__6C_gameFP7C_actor |
| 0x00306b60 | 248 | GLOBAL | C_game | DelNewActorFromAll | DelNewActorFromAll__6C_gameFP7C_actor |
| 0x00306c60 | 140 | GLOBAL | C_game | IsActorInActList | IsActorInActList__6C_gameFP7C_actor |
| 0x00306cf0 | 120 | GLOBAL | C_game | AddActorForFQL | AddActorForFQL__6C_gameFPCcPCci |
| 0x00306d70 | 1612 | GLOBAL | C_game | CreateActorsFQL | CreateActorsFQL__6C_gameFv |
| 0x003073c0 | 72 | WEAK | std::__vector_deleter<Q26C_game12S_enemy_talk,Q23std34allocator<Q26C_game12S_enemy_talk>> | clear | clear__Q23std84__vector_deleter<Q26C_game12S_enemy_talk,Q23std34allocator<Q26C_game12S_enemy_talk>>Fv |
| 0x00307410 | 72 | WEAK | std::__vector_deleter<Q26C_game13S_i_show_fade,Q23std35allocator<Q26C_game13S_i_show_fade>> | clear | clear__Q23std86__vector_deleter<Q26C_game13S_i_show_fade,Q23std35allocator<Q26C_game13S_i_show_fade>>Fv |
| 0x00307460 | 80 | WEAK | std::__vector_deleter<Q26C_game8S_i_show,Q23std29allocator<Q26C_game8S_i_show>> | clear | clear__Q23std74__vector_deleter<Q26C_game8S_i_show,Q23std29allocator<Q26C_game8S_i_show>>Fv |
| 0x003074b0 | 72 | WEAK | std::__vector_deleter<Q26C_game12S_driver_man,Q23std34allocator<Q26C_game12S_driver_man>> | clear | clear__Q23std84__vector_deleter<Q26C_game12S_driver_man,Q23std34allocator<Q26C_game12S_driver_man>>Fv |
| 0x00307500 | 80 | WEAK | std::__vector_deleter<Q26C_game12S_sound_play,Q23std34allocator<Q26C_game12S_sound_play>> | clear | clear__Q23std84__vector_deleter<Q26C_game12S_sound_play,Q23std34allocator<Q26C_game12S_sound_play>>Fv |
| 0x00307550 | 80 | WEAK | std::__vector_deleter<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>> | clear | clear__Q23std82__vector_deleter<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>>Fv |
| 0x003075a0 | 80 | WEAK | std::__vector_deleter<11S_semaphore,Q23std24allocator<11S_semaphore>> | clear | clear__Q23std64__vector_deleter<11S_semaphore,Q23std24allocator<11S_semaphore>>Fv |
| 0x003075f0 | 1052 | WEAK | std::__vector_imp<Q26C_game17S_invisible_actor,Q23std39allocator<Q26C_game17S_invisible_actor>,0> | insert | insert__Q23std92__vector_imp<Q26C_game17S_invisible_actor,Q23std39allocator<Q26C_game17S_invisible_actor>,0>FPQ26C_game17S_invisible_actorUiRCQ26C_game17S_invisible_actor |
| 0x00307a10 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std39allocator<Q26C_game17S_invisible_actor>,Ui,1> | swap | swap__Q310Metrowerks7details73compressed_pair_imp<Q23std39allocator<Q26C_game17S_invisible_actor>,Ui,1>FRQ310Metrowerks7details73compressed_pair_imp<Q23std39allocator<Q26C_game17S_invisible_actor>,Ui,1> |
| 0x00307a30 | 1300 | WEAK | std::__vector_imp<11S_semaphore,Q23std24allocator<11S_semaphore>,0> | insert | insert__Q23std62__vector_imp<11S_semaphore,Q23std24allocator<11S_semaphore>,0>FP11S_semaphoreUiRC11S_semaphore |
| 0x00307f50 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std24allocator<11S_semaphore>,Ui,1> | swap | swap__Q310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_semaphore>,Ui,1>FRQ310Metrowerks7details58compressed_pair_imp<Q23std24allocator<11S_semaphore>,Ui,1> |
| 0x00307f70 | 1284 | WEAK | std::__vector_imp<Q26C_game11S_RoadBlock,Q23std33allocator<Q26C_game11S_RoadBlock>,0> | insert | insert__Q23std80__vector_imp<Q26C_game11S_RoadBlock,Q23std33allocator<Q26C_game11S_RoadBlock>,0>FPQ26C_game11S_RoadBlockUiRCQ26C_game11S_RoadBlock |
| 0x00308480 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std33allocator<Q26C_game11S_RoadBlock>,Ui,1> | swap | swap__Q310Metrowerks7details67compressed_pair_imp<Q23std33allocator<Q26C_game11S_RoadBlock>,Ui,1>FRQ310Metrowerks7details67compressed_pair_imp<Q23std33allocator<Q26C_game11S_RoadBlock>,Ui,1> |
| 0x003084a0 | 40 | 13 | std::__vector_imp<P7C_actor,Q23std20allocator<P7C_actor>,1> | operator= | __as__Q23std54__vector_imp<P7C_actor,Q23std20allocator<P7C_actor>,1>FRCQ23std54__vector_imp<P7C_actor,Q23std20allocator<P7C_actor>,1> |
| 0x003084d0 | 8 | 13 | Metrowerks::compile_assert<1> | {ctor} | __ct__Q210Metrowerks17compile_assert<1>FRCQ210Metrowerks17compile_assert<1> |
| 0x003084e0 | 1172 | WEAK | std::__vector_imp<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>,0> | insert | insert__Q23std80__vector_imp<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>,0>FPQ26C_game11S_TelephoneUiRCQ26C_game11S_Telephone |
| 0x00308980 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std33allocator<Q26C_game11S_Telephone>,Ui,1> | swap | swap__Q310Metrowerks7details67compressed_pair_imp<Q23std33allocator<Q26C_game11S_Telephone>,Ui,1>FRQ310Metrowerks7details67compressed_pair_imp<Q23std33allocator<Q26C_game11S_Telephone>,Ui,1> |
| 0x003089a0 | 1388 | WEAK | std::__vector_imp<Q26C_game13S_shadow_item,Q23std35allocator<Q26C_game13S_shadow_item>,0> | insert | insert__Q23std84__vector_imp<Q26C_game13S_shadow_item,Q23std35allocator<Q26C_game13S_shadow_item>,0>FPQ26C_game13S_shadow_itemUiRCQ26C_game13S_shadow_item |
| 0x00308f10 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std35allocator<Q26C_game13S_shadow_item>,Ui,1> | swap | swap__Q310Metrowerks7details69compressed_pair_imp<Q23std35allocator<Q26C_game13S_shadow_item>,Ui,1>FRQ310Metrowerks7details69compressed_pair_imp<Q23std35allocator<Q26C_game13S_shadow_item>,Ui,1> |
| 0x00308f30 | 64 | 13 | C_game::S_shadow_item | operator= | __as__Q26C_game13S_shadow_itemFRCQ26C_game13S_shadow_item |
| 0x00308f70 | 1364 | WEAK | std::__vector_imp<Q26C_game12S_sound_play,Q23std34allocator<Q26C_game12S_sound_play>,0> | insert | insert__Q23std82__vector_imp<Q26C_game12S_sound_play,Q23std34allocator<Q26C_game12S_sound_play>,0>FPQ26C_game12S_sound_playUiRCQ26C_game12S_sound_play |
| 0x003094d0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std34allocator<Q26C_game12S_sound_play>,Ui,1> | swap | swap__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q26C_game12S_sound_play>,Ui,1>FRQ310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q26C_game12S_sound_play>,Ui,1> |
| 0x003094f0 | 1588 | WEAK | std::__vector_imp<Q26C_game16S_npc_kill_event,Q23std38allocator<Q26C_game16S_npc_kill_event>,0> | insert | insert__Q23std90__vector_imp<Q26C_game16S_npc_kill_event,Q23std38allocator<Q26C_game16S_npc_kill_event>,0>FPQ26C_game16S_npc_kill_eventUiRCQ26C_game16S_npc_kill_event |
| 0x00309b30 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std38allocator<Q26C_game16S_npc_kill_event>,Ui,1> | swap | swap__Q310Metrowerks7details72compressed_pair_imp<Q23std38allocator<Q26C_game16S_npc_kill_event>,Ui,1>FRQ310Metrowerks7details72compressed_pair_imp<Q23std38allocator<Q26C_game16S_npc_kill_event>,Ui,1> |
| 0x00309b50 | 1180 | WEAK | std::__vector_imp<Q26C_game12S_driver_man,Q23std34allocator<Q26C_game12S_driver_man>,0> | insert | insert__Q23std82__vector_imp<Q26C_game12S_driver_man,Q23std34allocator<Q26C_game12S_driver_man>,0>FPQ26C_game12S_driver_manUiRCQ26C_game12S_driver_man |
| 0x00309ff0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std34allocator<Q26C_game12S_driver_man>,Ui,1> | swap | swap__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q26C_game12S_driver_man>,Ui,1>FRQ310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q26C_game12S_driver_man>,Ui,1> |
| 0x0030a010 | 1300 | WEAK | std::__vector_imp<Q26C_game8S_i_show,Q23std29allocator<Q26C_game8S_i_show>,0> | insert | insert__Q23std72__vector_imp<Q26C_game8S_i_show,Q23std29allocator<Q26C_game8S_i_show>,0>FPQ26C_game8S_i_showUiRCQ26C_game8S_i_show |
| 0x0030a530 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std29allocator<Q26C_game8S_i_show>,Ui,1> | swap | swap__Q310Metrowerks7details63compressed_pair_imp<Q23std29allocator<Q26C_game8S_i_show>,Ui,1>FRQ310Metrowerks7details63compressed_pair_imp<Q23std29allocator<Q26C_game8S_i_show>,Ui,1> |
| 0x0030a550 | 1196 | WEAK | std::__vector_imp<Q26C_game13S_i_show_fade,Q23std35allocator<Q26C_game13S_i_show_fade>,0> | insert | insert__Q23std84__vector_imp<Q26C_game13S_i_show_fade,Q23std35allocator<Q26C_game13S_i_show_fade>,0>FPQ26C_game13S_i_show_fadeUiRCQ26C_game13S_i_show_fade |
| 0x0030aa00 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std35allocator<Q26C_game13S_i_show_fade>,Ui,1> | swap | swap__Q310Metrowerks7details69compressed_pair_imp<Q23std35allocator<Q26C_game13S_i_show_fade>,Ui,1>FRQ310Metrowerks7details69compressed_pair_imp<Q23std35allocator<Q26C_game13S_i_show_fade>,Ui,1> |
| 0x0030aa20 | 1052 | WEAK | std::__vector_imp<Q26C_game12S_enemy_talk,Q23std34allocator<Q26C_game12S_enemy_talk>,0> | insert | insert__Q23std82__vector_imp<Q26C_game12S_enemy_talk,Q23std34allocator<Q26C_game12S_enemy_talk>,0>FPQ26C_game12S_enemy_talkUiRCQ26C_game12S_enemy_talk |
| 0x0030ae40 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std34allocator<Q26C_game12S_enemy_talk>,Ui,1> | swap | swap__Q310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q26C_game12S_enemy_talk>,Ui,1>FRQ310Metrowerks7details68compressed_pair_imp<Q23std34allocator<Q26C_game12S_enemy_talk>,Ui,1> |
| 0x0030ae60 | 1052 | WEAK | std::__vector_imp<Q26C_game13S_scream_area,Q23std35allocator<Q26C_game13S_scream_area>,0> | insert | insert__Q23std84__vector_imp<Q26C_game13S_scream_area,Q23std35allocator<Q26C_game13S_scream_area>,0>FPQ26C_game13S_scream_areaUiRCQ26C_game13S_scream_area |
| 0x0030b280 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std35allocator<Q26C_game13S_scream_area>,Ui,1> | swap | swap__Q310Metrowerks7details69compressed_pair_imp<Q23std35allocator<Q26C_game13S_scream_area>,Ui,1>FRQ310Metrowerks7details69compressed_pair_imp<Q23std35allocator<Q26C_game13S_scream_area>,Ui,1> |
| 0x0030b2a0 | 1436 | WEAK | std::__vector_imp<Q26C_game15S_save_game_fql,Q23std37allocator<Q26C_game15S_save_game_fql>,0> | insert | insert__Q23std88__vector_imp<Q26C_game15S_save_game_fql,Q23std37allocator<Q26C_game15S_save_game_fql>,0>FPQ26C_game15S_save_game_fqlUiRCQ26C_game15S_save_game_fql |
| 0x0030b840 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std37allocator<Q26C_game15S_save_game_fql>,Ui,1> | swap | swap__Q310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q26C_game15S_save_game_fql>,Ui,1>FRQ310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q26C_game15S_save_game_fql>,Ui,1> |
| 0x0030b860 | 1052 | WEAK | std::__vector_imp<9S_car_snd,Q23std21allocator<9S_car_snd>,0> | insert | insert__Q23std56__vector_imp<9S_car_snd,Q23std21allocator<9S_car_snd>,0>FP9S_car_sndUiRC9S_car_snd |
| 0x0030bc80 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std21allocator<9S_car_snd>,Ui,1> | swap | swap__Q310Metrowerks7details55compressed_pair_imp<Q23std21allocator<9S_car_snd>,Ui,1>FRQ310Metrowerks7details55compressed_pair_imp<Q23std21allocator<9S_car_snd>,Ui,1> |
| 0x0030bca0 | 72 | WEAK | std::__vector_deleter<Q26C_game17S_invisible_actor,Q23std39allocator<Q26C_game17S_invisible_actor>> | clear | clear__Q23std94__vector_deleter<Q26C_game17S_invisible_actor,Q23std39allocator<Q26C_game17S_invisible_actor>>Fv |
| 0x0030bcf0 | 124 | WEAK | std::__vector_deleter<Q26C_game11S_RoadBlock,Q23std33allocator<Q26C_game11S_RoadBlock>> | clear | clear__Q23std82__vector_deleter<Q26C_game11S_RoadBlock,Q23std33allocator<Q26C_game11S_RoadBlock>>Fv |
| 0x0030bd70 | 72 | WEAK | std::__vector_deleter<Q26C_game13S_shadow_temp,Q23std35allocator<Q26C_game13S_shadow_temp>> | clear | clear__Q23std86__vector_deleter<Q26C_game13S_shadow_temp,Q23std35allocator<Q26C_game13S_shadow_temp>>Fv |
| 0x0030bdc0 | 80 | WEAK | std::__vector_deleter<Q26C_game13S_shadow_item,Q23std35allocator<Q26C_game13S_shadow_item>> | clear | clear__Q23std86__vector_deleter<Q26C_game13S_shadow_item,Q23std35allocator<Q26C_game13S_shadow_item>>Fv |
| 0x0030be10 | 80 | WEAK | std::__vector_deleter<Q26C_game16S_npc_kill_event,Q23std38allocator<Q26C_game16S_npc_kill_event>> | clear | clear__Q23std92__vector_deleter<Q26C_game16S_npc_kill_event,Q23std38allocator<Q26C_game16S_npc_kill_event>>Fv |
| 0x0030be60 | 80 | WEAK | std::__vector_deleter<Q26C_game12S_find_actor,Q23std34allocator<Q26C_game12S_find_actor>> | clear | clear__Q23std84__vector_deleter<Q26C_game12S_find_actor,Q23std34allocator<Q26C_game12S_find_actor>>Fv |
| 0x0030beb0 | 72 | WEAK | std::__vector_deleter<Q26C_game13S_scream_area,Q23std35allocator<Q26C_game13S_scream_area>> | clear | clear__Q23std86__vector_deleter<Q26C_game13S_scream_area,Q23std35allocator<Q26C_game13S_scream_area>>Fv |
| 0x0030bf00 | 80 | WEAK | std::__vector_deleter<Q26C_game15S_save_game_fql,Q23std37allocator<Q26C_game15S_save_game_fql>> | clear | clear__Q23std90__vector_deleter<Q26C_game15S_save_game_fql,Q23std37allocator<Q26C_game15S_save_game_fql>>Fv |
| 0x0030bf50 | 104 | WEAK | std::__vector_deleter<9S_car_snd,Q23std21allocator<9S_car_snd>> | {dtor} | __dt__Q23std58__vector_deleter<9S_car_snd,Q23std21allocator<9S_car_snd>>Fv |
| 0x0030bfc0 | 104 | WEAK | std::__vector_deleter<Q26C_game17S_invisible_actor,Q23std39allocator<Q26C_game17S_invisible_actor>> | {dtor} | __dt__Q23std94__vector_deleter<Q26C_game17S_invisible_actor,Q23std39allocator<Q26C_game17S_invisible_actor>>Fv |
| 0x0030c030 | 104 | WEAK | std::__vector_deleter<11S_semaphore,Q23std24allocator<11S_semaphore>> | {dtor} | __dt__Q23std64__vector_deleter<11S_semaphore,Q23std24allocator<11S_semaphore>>Fv |
| 0x0030c0a0 | 104 | WEAK | std::__vector_deleter<Q26C_game11S_RoadBlock,Q23std33allocator<Q26C_game11S_RoadBlock>> | {dtor} | __dt__Q23std82__vector_deleter<Q26C_game11S_RoadBlock,Q23std33allocator<Q26C_game11S_RoadBlock>>Fv |
| 0x0030c110 | 104 | WEAK | std::__vector_deleter<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>> | {dtor} | __dt__Q23std82__vector_deleter<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>>Fv |
| 0x0030c180 | 104 | WEAK | std::__vector_deleter<Q26C_game13S_shadow_temp,Q23std35allocator<Q26C_game13S_shadow_temp>> | {dtor} | __dt__Q23std86__vector_deleter<Q26C_game13S_shadow_temp,Q23std35allocator<Q26C_game13S_shadow_temp>>Fv |
| 0x0030c1f0 | 104 | WEAK | std::__vector_deleter<Q26C_game13S_shadow_item,Q23std35allocator<Q26C_game13S_shadow_item>> | {dtor} | __dt__Q23std86__vector_deleter<Q26C_game13S_shadow_item,Q23std35allocator<Q26C_game13S_shadow_item>>Fv |
| 0x0030c260 | 104 | WEAK | std::__vector_deleter<Q26C_game12S_sound_play,Q23std34allocator<Q26C_game12S_sound_play>> | {dtor} | __dt__Q23std84__vector_deleter<Q26C_game12S_sound_play,Q23std34allocator<Q26C_game12S_sound_play>>Fv |
| 0x0030c2d0 | 104 | WEAK | std::__vector_deleter<Q26C_game16S_npc_kill_event,Q23std38allocator<Q26C_game16S_npc_kill_event>> | {dtor} | __dt__Q23std92__vector_deleter<Q26C_game16S_npc_kill_event,Q23std38allocator<Q26C_game16S_npc_kill_event>>Fv |
| 0x0030c340 | 104 | WEAK | std::__vector_deleter<Q26C_game12S_driver_man,Q23std34allocator<Q26C_game12S_driver_man>> | {dtor} | __dt__Q23std84__vector_deleter<Q26C_game12S_driver_man,Q23std34allocator<Q26C_game12S_driver_man>>Fv |
| 0x0030c3b0 | 104 | WEAK | std::__vector_deleter<Q26C_game8S_i_show,Q23std29allocator<Q26C_game8S_i_show>> | {dtor} | __dt__Q23std74__vector_deleter<Q26C_game8S_i_show,Q23std29allocator<Q26C_game8S_i_show>>Fv |
| 0x0030c420 | 104 | WEAK | std::__vector_deleter<Q26C_game13S_i_show_fade,Q23std35allocator<Q26C_game13S_i_show_fade>> | {dtor} | __dt__Q23std86__vector_deleter<Q26C_game13S_i_show_fade,Q23std35allocator<Q26C_game13S_i_show_fade>>Fv |
| 0x0030c490 | 104 | WEAK | std::__vector_deleter<Q26C_game12S_enemy_talk,Q23std34allocator<Q26C_game12S_enemy_talk>> | {dtor} | __dt__Q23std84__vector_deleter<Q26C_game12S_enemy_talk,Q23std34allocator<Q26C_game12S_enemy_talk>>Fv |
| 0x0030c500 | 104 | WEAK | std::__vector_deleter<Q26C_game12S_find_actor,Q23std34allocator<Q26C_game12S_find_actor>> | {dtor} | __dt__Q23std84__vector_deleter<Q26C_game12S_find_actor,Q23std34allocator<Q26C_game12S_find_actor>>Fv |
| 0x0030c570 | 104 | WEAK | std::__vector_deleter<Q26C_game13S_scream_area,Q23std35allocator<Q26C_game13S_scream_area>> | {dtor} | __dt__Q23std86__vector_deleter<Q26C_game13S_scream_area,Q23std35allocator<Q26C_game13S_scream_area>>Fv |
| 0x0030c5e0 | 104 | WEAK | std::__vector_deleter<Q26C_game15S_save_game_fql,Q23std37allocator<Q26C_game15S_save_game_fql>> | {dtor} | __dt__Q23std90__vector_deleter<Q26C_game15S_save_game_fql,Q23std37allocator<Q26C_game15S_save_game_fql>>Fv |
| 0x0030c650 | 72 | WEAK | std::__vector_deleter<9S_car_snd,Q23std21allocator<9S_car_snd>> | clear | clear__Q23std58__vector_deleter<9S_car_snd,Q23std21allocator<9S_car_snd>>Fv |
| 0x0030c6a0 | 96 | GLOBAL | ai_character_log | NewContact | NewContact__16ai_character_logFP7C_actorUi |
| 0x0030c700 | 60 | GLOBAL | ai_logs | Init | Init__7ai_logsFPv |
| 0x0030c740 | 1132 | GLOBAL | ai_logs | Tick | Tick__7ai_logsFv |
| 0x0030cbb0 | 1552 | GLOBAL | ai_logs | Event | Event__7ai_logsFP7C_actorUiPv |
| 0x0030d1c0 | 156 | GLOBAL | ai_logs | GetAimingWeight | GetAimingWeight__7ai_logsFP16ai_character_log |
| 0x0030d260 | 156 | GLOBAL | ai_logs | GetShootingWeight | GetShootingWeight__7ai_logsFP16ai_character_log |
| 0x0030d300 | 720 | GLOBAL | ai_logs | ComputeAttitude | ComputeAttitude__7ai_logsFP16ai_character_log |
| 0x0030d5d0 | 244 | GLOBAL | ai_character_log | EventHit | EventHit__16ai_character_logFP12ai_event_hit |
| 0x0030d6d0 | 120 | GLOBAL | ai_character_log | EventEnemyHit | EventEnemyHit__16ai_character_logFP12ai_event_hit |
| 0x0030d750 | 168 | GLOBAL | ai_character_log | EventFriendHit | EventFriendHit__16ai_character_logFPf |
| 0x0030d800 | 164 | GLOBAL | ai_character_log | EventVisualCon | EventVisualCon__16ai_character_logFb |
| 0x0030d8b0 | 100 | GLOBAL | ai_character_log | EventSoundCon | EventSoundCon__16ai_character_logFv |
| 0x0030d920 | 116 | GLOBAL | ai_logs | hasLog | hasLog__7ai_logsFP7C_actorPUi |
| 0x0030d9a0 | 92 | GLOBAL | ai_logs | GetLog | GetLog__7ai_logsFP7C_actor |
| 0x0030da00 | 168 | GLOBAL | ai_logs | GetMaxAngryCharacter | GetMaxAngryCharacter__7ai_logsFv |
| 0x0030dab0 | 168 | GLOBAL | ai_logs | GetMaxFearCharacter | GetMaxFearCharacter__7ai_logsFv |
| 0x0030db60 | 168 | GLOBAL | ai_logs | GetMaxDeathCharacter | GetMaxDeathCharacter__7ai_logsFv |
| 0x0030dc10 | 180 | GLOBAL | ai_logs | GetMaxOhrozeni | GetMaxOhrozeni__7ai_logsFv |
| 0x0030dcd0 | 116 | GLOBAL | ai_logs | Destroy | Destroy__7ai_logsFP7C_actor |
| 0x0030dd50 | 656 | GLOBAL | ai_logs | LooseDistant | LooseDistant__7ai_logsFv |
| 0x0030dfe0 | 8 | 13 | std::__vector_deleter<16ai_character_log,Q23std29allocator<16ai_character_log>> | data | data__Q23std74__vector_deleter<16ai_character_log,Q23std29allocator<16ai_character_log>>Fv |
| 0x0030dff0 | 216 | WEAK | std::__vector_imp<16ai_character_log,Q23std29allocator<16ai_character_log>,0> | erase | erase__Q23std72__vector_imp<16ai_character_log,Q23std29allocator<16ai_character_log>,0>FP16ai_character_log |
| 0x0030e0d0 | 176 | 13 | ai_character_log | operator= | __as__16ai_character_logFRC16ai_character_log |
| 0x0030e180 | 1292 | WEAK | std::__vector_imp<16ai_character_log,Q23std29allocator<16ai_character_log>,0> | insert | insert__Q23std72__vector_imp<16ai_character_log,Q23std29allocator<16ai_character_log>,0>FP16ai_character_logUiRC16ai_character_log |
| 0x0030e690 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std29allocator<16ai_character_log>,Ui,1> | swap | swap__Q310Metrowerks7details63compressed_pair_imp<Q23std29allocator<16ai_character_log>,Ui,1>FRQ310Metrowerks7details63compressed_pair_imp<Q23std29allocator<16ai_character_log>,Ui,1> |
| 0x0030e6b0 | 104 | WEAK | std::__vector_deleter<16ai_character_log,Q23std29allocator<16ai_character_log>> | {dtor} | __dt__Q23std74__vector_deleter<16ai_character_log,Q23std29allocator<16ai_character_log>>Fv |
| 0x0030e720 | 72 | GLOBAL | ai_sensors | hasVisualCon | hasVisualCon__10ai_sensorsFP7C_actor |
| 0x0030e770 | 164 | GLOBAL | ai_sensors | hasSoundCon | hasSoundCon__10ai_sensorsFP7C_actorUib |
| 0x0030e820 | 72 | GLOBAL | ai_sensors | hasSmellCon | hasSmellCon__10ai_sensorsFP7C_actor |
| 0x0030e870 | 196 | GLOBAL | ai_sensors | NewVisualCon | NewVisualCon__10ai_sensorsFP17ai_visual_contact |
| 0x0030e940 | 124 | GLOBAL | ai_sensors | UpdateVisualCon | UpdateVisualCon__10ai_sensorsFiP17ai_visual_contact |
| 0x0030e9c0 | 376 | GLOBAL | ai_sensors | CancelVisualCon | CancelVisualCon__10ai_sensorsFP7C_actorb |
| 0x0030eb40 | 368 | GLOBAL | ai_sensors | UpdateVIS | UpdateVIS__10ai_sensorsFUiUi |
| 0x0030ecb0 | 252 | GLOBAL | ai_sensors | NewSoundCon | NewSoundCon__10ai_sensorsFP16ai_sound_contact |
| 0x0030edb0 | 128 | GLOBAL | ai_sensors | UpdateSoundCon | UpdateSoundCon__10ai_sensorsFiP16ai_sound_contact |
| 0x0030ee30 | 88 | GLOBAL | ai_sensors | CancelSoundCon | CancelSoundCon__10ai_sensorsFP7C_actorUi |
| 0x0030ee90 | 368 | GLOBAL | ai_sensors | UpdateSND | UpdateSND__10ai_sensorsFUiUi |
| 0x0030f000 | 228 | GLOBAL | ai_sensors | NewSmellCon | NewSmellCon__10ai_sensorsFP16ai_smell_contact |
| 0x0030f0f0 | 208 | GLOBAL | ai_sensors | UpdateSmellCon | UpdateSmellCon__10ai_sensorsFiP16ai_smell_contact |
| 0x0030f1c0 | 140 | GLOBAL | ai_sensors | CancelSmellCon | CancelSmellCon__10ai_sensorsFP7C_actor |
| 0x0030f250 | 368 | GLOBAL | ai_sensors | UpdateSMELL | UpdateSMELL__10ai_sensorsFUiUi |
| 0x0030f3c0 | 156 | GLOBAL | ai_sensors | Init | Init__10ai_sensorsFPv |
| 0x0030f460 | 244 | GLOBAL | ai_sensors | Destroy | Destroy__10ai_sensorsFv |
| 0x0030f560 | 136 | GLOBAL | ai_sensors | isVisible | isVisible__10ai_sensorsFP7C_actor |
| 0x0030f5f0 | 212 | GLOBAL | ai_sensors | isHearable | isHearable__10ai_sensorsFP7C_actorUib |
| 0x0030f6d0 | 172 | GLOBAL | ai_sensors | isSmellable | isSmellable__10ai_sensorsFP7C_actorUi |
| 0x0030f780 | 484 | GLOBAL | ai_sensors | Sniff | Sniff__10ai_sensorsFP7C_actorP8S_vectorP8S_vectorPUiPUi |
| 0x0030f970 | 220 | GLOBAL | ai_sensors | Hear | Hear__10ai_sensorsFP7C_actorP8S_vectorPUiUi |
| 0x0030fa50 | 48 | GLOBAL | ai_sensors | AnalyzeBlob | AnalyzeBlob__10ai_sensorsFP7ai_blobP16ai_smell_contact |
| 0x0030fa80 | 196 | GLOBAL | ai_vis_logs | Init | Init__11ai_vis_logsFPv |
| 0x0030fb50 | 256 | GLOBAL | ai_vis_logs | AddEntity | AddEntity__11ai_vis_logsFPvfb |
| 0x0030fc50 | 144 | GLOBAL | ai_vis_logs | DelEntity | DelEntity__11ai_vis_logsFPvb |
| 0x0030fce0 | 4008 | GLOBAL | ai_vis_logs | Tick | Tick__11ai_vis_logsFv |
| 0x00310c90 | 1648 | GLOBAL | ai_vis_logs | TickCorpses | TickCorpses__11ai_vis_logsFv |
| 0x00311300 | 332 | GLOBAL | C_entity | {ctor} | __ct__8C_entityFv |
| 0x00311450 | 20 | 13 | std::__vector_deleter<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>> | {ctor} | __ct__Q23std74__vector_deleter<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>>Fv |
| 0x00311470 | 20 | 13 | std::__vector_deleter<16ai_character_log,Q23std29allocator<16ai_character_log>> | {ctor} | __ct__Q23std74__vector_deleter<16ai_character_log,Q23std29allocator<16ai_character_log>>Fv |
| 0x00311490 | 440 | GLOBAL | C_entity | {dtor} | __dt__8C_entityFv |
| 0x00311650 | 32 | GLOBAL | C_entity | Init | Init__8C_entityFP9I3D_frame |
| 0x00311670 | 68 | GLOBAL | C_entity | DuplicateFrom | DuplicateFrom__8C_entityFP7C_actor |
| 0x003116c0 | 1296 | GLOBAL | C_entity | GameInit | GameInit__8C_entityFv |
| 0x00311bd0 | 208 | GLOBAL | C_entity | Reset | Reset__8C_entityFv |
| 0x00311ca0 | 100 | GLOBAL | C_entity | ResetState | ResetState__8C_entityFv |
| 0x00311d10 | 200 | GLOBAL | C_entity | GameDone | GameDone__8C_entityFv |
| 0x00311de0 | 116 | GLOBAL | C_entity | SaveData | SaveData__8C_entityFP7C_chunk |
| 0x00311e60 | 196 | GLOBAL | C_entity | LoadData | LoadData__8C_entityFP7C_chunk |
| 0x00311f30 | 8 | GLOBAL | C_entity | Collision | Collision__8C_entityFP7C_actorRC8S_vectorRC8S_vectorRC8S_vectorfP9I3D_frame |
| 0x00311f40 | 756 | GLOBAL | C_entity | Hit | Hit__8C_entityF10E_hit_typeRC8S_vectorRC8S_vectorRC8S_vectorfP7C_actorUiP9I3D_frame |
| 0x00312240 | 188 | GLOBAL | C_entity | cbHitInfo | cbHitInfo__8C_entityF10E_hit_typefP7C_actor |
| 0x00312300 | 11396 | GLOBAL | C_entity | AI | AI__8C_entityFUi |
| 0x00314f90 | 176 | 13 | ai_modifier | Tick | Tick__11ai_modifierFUi |
| 0x00315040 | 45488 | GLOBAL | C_entity | actionTick | actionTick__8C_entityFP9ai_actionUiUiPUi |
| 0x003201f0 | 59548 | GLOBAL | C_entity | actionTick_2 | actionTick_2__8C_entityFP9ai_actionUiUiPUi |
| 0x0032ea90 | 12 | 13 | C_traffic_car | DajDevatemuTyZkurvenyZmrdyAtFurtNevotravuje | DajDevatemuTyZkurvenyZmrdyAtFurtNevotravuje__13C_traffic_carFv |
| 0x0032eaa0 | 1156 | GLOBAL | C_entity | actionsTick | actionsTick__8C_entityFUi |
| 0x0032ef30 | 8 | GLOBAL | C_entity | NetDirect | NetDirect__8C_entityFUi |
| 0x0032ef40 | 1252 | GLOBAL | C_entity | Update | Update__8C_entityFUi |
| 0x0032f430 | 324 | GLOBAL | C_entity | Death | Death__8C_entityFv |
| 0x0032f580 | 13608 | GLOBAL | C_entity | DecodeParams | DecodeParams__8C_entityFPcRQ29C_program13S_instruction |
| 0x00332ab0 | 348 | GLOBAL | C_entity | DecodeInstruction | DecodeInstruction__8C_entityFPcRi |
| 0x00332c10 | 2712 | GLOBAL | C_entity | DecompileCommand | DecompileCommand__8C_entityFiPc |
| 0x003336b0 | 22332 | GLOBAL | C_entity | Process | Process__8C_entityFUi |
| 0x00338df0 | 1924 | GLOBAL | C_entity | scriptTick | scriptTick__8C_entityFUi |
| 0x00339580 | 44 | GLOBAL | C_entity | GetVisibleRange | GetVisibleRange__8C_entityFv |
| 0x003395b0 | 8 | GLOBAL | C_entity | GetHearQuality | GetHearQuality__8C_entityFv |
| 0x003395c0 | 12 | GLOBAL | C_entity | GetHeadDir | GetHeadDir__8C_entityFv |
| 0x003395d0 | 12 | GLOBAL | C_entity | GetHeadPos | GetHeadPos__8C_entityFv |
| 0x003395e0 | 192 | GLOBAL | C_entity | ScriptSwitch | ScriptSwitch__8C_entityFv |
| 0x003396a0 | 564 | GLOBAL | C_entity | ForceAI | ForceAI__8C_entityFUiUiUif |
| 0x003398e0 | 284 | GLOBAL | C_entity | isFakeScript | isFakeScript__8C_entityFv |
| 0x00339a00 | 332 | GLOBAL | C_entity | ForceScript | ForceScript__8C_entityFbb |
| 0x00339b50 | 68 | GLOBAL | C_entity | CallBack | CallBack__8C_entityFQ29C_program11E_cb_action |
| 0x00339ba0 | 160 | GLOBAL | C_entity | ChangeState | ChangeState__8C_entityFv |
| 0x00339c40 | 3192 | GLOBAL | C_entity | Event | Event__8C_entityFP7C_actorUiPv |
| 0x0033a8c0 | 236 | GLOBAL | ai_action_manager | Clear | Clear__17ai_action_managerFb |
| 0x0033a9b0 | 20 | GLOBAL | ai_action_manager | GenUID | GenUID__17ai_action_managerFv |
| 0x0033a9d0 | 2164 | GLOBAL | ai_action_manager | DestroyAction | DestroyAction__17ai_action_managerFP9ai_action |
| 0x0033b250 | 104 | 13 | gif::vector<12ai_path_node,4> | {dtor} | __dt__Q23gif24vector<12ai_path_node,4>Fv |
| 0x0033b2c0 | 104 | 13 | gif::vector<16ai_path_specnode,2> | {dtor} | __dt__Q23gif28vector<16ai_path_specnode,2>Fv |
| 0x0033b330 | 104 | 13 | gif::vector<16ai_actor_evading,1> | {dtor} | __dt__Q23gif28vector<16ai_actor_evading,1>Fv |
| 0x0033b3a0 | 740 | GLOBAL | ai_action_manager | InvalidatePath | InvalidatePath__17ai_action_managerFP9ai_actionP7C_actor |
| 0x0033b690 | 224 | GLOBAL | ai_action_manager | Invalidate | Invalidate__17ai_action_managerFP7C_actor |
| 0x0033b770 | 264 | GLOBAL | ai_action_manager | NewMove | NewMove__17ai_action_managerFR8S_vectorUiUiUifP7ai_pathUi |
| 0x0033b880 | 248 | GLOBAL | ai_action_manager | NewFollow | NewFollow__17ai_action_managerFP7C_actorfUiUiP7ai_pathUi |
| 0x0033b980 | 152 | GLOBAL | ai_action_manager | NewTurnTo | NewTurnTo__17ai_action_managerFP7C_actorUi |
| 0x0033ba20 | 220 | GLOBAL | ai_action_manager | NewFire | NewFire__17ai_action_managerFP7C_actorfUiUi |
| 0x0033bb00 | 256 | GLOBAL | ai_action_manager | NewFollowLink | NewFollowLink__17ai_action_managerFP7C_humanUiUifP7ai_pathUi |
| 0x0033bc00 | 184 | GLOBAL | ai_action_manager | NewBlockedWay | NewBlockedWay__17ai_action_managerFUiUiUi |
| 0x0033bcc0 | 304 | GLOBAL | ai_action_manager | NewRunawayDist | NewRunawayDist__17ai_action_managerFP7C_actorfUiUiUiP7ai_pathUi |
| 0x0033bdf0 | 204 | GLOBAL | ai_action_manager | NewSideLook | NewSideLook__17ai_action_managerFR8S_vectorfUiUi |
| 0x0033bec0 | 412 | GLOBAL | ai_action_manager | NewBackLook | NewBackLook__17ai_action_managerFR8S_vectorUibUi |
| 0x0033c060 | 164 | GLOBAL | ai_action_manager | NewUnLink | NewUnLink__17ai_action_managerFUiUi |
| 0x0033c110 | 312 | GLOBAL | ai_action_manager | NewRunCloserTo | NewRunCloserTo__17ai_action_managerFP7C_actorffUiUiP7ai_pathUi |
| 0x0033c250 | 184 | GLOBAL | ai_action_manager | NewFreeze | NewFreeze__17ai_action_managerFUiUiUi |
| 0x0033c310 | 280 | GLOBAL | ai_action_manager | NewMoveToCar | NewMoveToCar__17ai_action_managerFP5C_carffUiUiUiP7ai_pathUi |
| 0x0033c430 | 156 | GLOBAL | ai_action_manager | NewStepAway | NewStepAway__17ai_action_managerFP7C_actorUi |
| 0x0033c4d0 | 264 | GLOBAL | ai_action_manager | NewStrafe | NewStrafe__17ai_action_managerFP7C_actorUiUifUi |
| 0x0033c5e0 | 168 | GLOBAL | ai_action_manager | NewJump | NewJump__17ai_action_managerFUiUi |
| 0x0033c690 | 216 | GLOBAL | ai_action_manager | NewCornerLook | NewCornerLook__17ai_action_managerFP7C_actorUiUiQ27C_human13E_corner_viewUi |
| 0x0033c770 | 216 | GLOBAL | ai_action_manager | NewCornerCoverFire | NewCornerCoverFire__17ai_action_managerFP7C_actorUiUiQ27C_human13E_corner_viewUi |
| 0x0033c850 | 216 | GLOBAL | ai_action_manager | NewCornerFire | NewCornerFire__17ai_action_managerFP7C_actorUiUiQ27C_human13E_corner_viewUi |
| 0x0033c930 | 188 | GLOBAL | ai_action_manager | NewCornerTakePos | NewCornerTakePos__17ai_action_managerFP7C_actorP13ai_seg_cornerUi |
| 0x0033c9f0 | 372 | GLOBAL | ai_action_manager | NewCornerFight | NewCornerFight__17ai_action_managerFP7C_actorP19ai_corner_hide_infoUi |
| 0x0033cb70 | 536 | GLOBAL | ai_action_manager | NewCrouchFight | NewCrouchFight__17ai_action_managerFP7C_actorP7C_actorfUiUi |
| 0x0033cd90 | 476 | GLOBAL | ai_action_manager | NewNormalFight | NewNormalFight__17ai_action_managerFP7C_actorP7C_actorfUiUi |
| 0x0033cf70 | 256 | GLOBAL | ai_action_manager | NewBarricadeFight | NewBarricadeFight__17ai_action_managerFP7C_actorP7C_actorffUi |
| 0x0033d070 | 228 | GLOBAL | ai_action_manager | NewFollowInRail | NewFollowInRail__17ai_action_managerFP7C_humanP7C_humanUi |
| 0x0033d160 | 348 | GLOBAL | ai_action_manager | NewPoliceAction | NewPoliceAction__17ai_action_managerFP7C_actorUifUiUi |
| 0x0033d2c0 | 260 | GLOBAL | ai_action_manager | NewSearch | NewSearch__17ai_action_managerFP7C_actorfUiUi |
| 0x0033d3d0 | 324 | GLOBAL | ai_action_manager | NewKickDriver | NewKickDriver__17ai_action_managerFP7C_humanffUiP7ai_pathUi |
| 0x0033d520 | 760 | GLOBAL | ai_action_manager | NewHostileCoverage | NewHostileCoverage__17ai_action_managerFP7C_actorUiPQ23std46vector<P7C_actor,Q23std20allocator<P7C_actor>>Ui |
| 0x0033d820 | 172 | GLOBAL | ai_action_manager | NewCarJump | NewCarJump__17ai_action_managerFUiUi |
| 0x0033d8d0 | 212 | GLOBAL | ai_action_manager | NewCarStop | NewCarStop__17ai_action_managerFP7C_actorP5C_carUiR8S_vectorUi |
| 0x0033d9b0 | 212 | GLOBAL | ai_action_manager | NewCarAim | NewCarAim__17ai_action_managerFP7C_actorP5C_carUiR8S_vectorUi |
| 0x0033da90 | 836 | GLOBAL | ai_action_manager | NewPoliceCall | NewPoliceCall__17ai_action_managerFUiUiUiP7ai_pathUi |
| 0x0033dde0 | 196 | GLOBAL | ai_action_manager | NewStrafeHim | NewStrafeHim__17ai_action_managerFP7C_humanUiUi |
| 0x0033deb0 | 204 | GLOBAL | ai_action_manager | NewStrafeBeware | NewStrafeBeware__17ai_action_managerFP12ai_fire_areaUiUi |
| 0x0033df80 | 72 | GLOBAL | ai_action_manager | AddJob | AddJob__17ai_action_managerFP9ai_action |
| 0x0033dfd0 | 84 | GLOBAL | ai_action_manager | NewJob | NewJob__17ai_action_managerFP9ai_action |
| 0x0033e030 | 76 | GLOBAL | ai_action_manager | AddModifier | AddModifier__17ai_action_managerFP9ai_action |
| 0x0033e080 | 136 | GLOBAL | ai_action_manager | NewModifier | NewModifier__17ai_action_managerFP9ai_action |
| 0x0033e110 | 76 | GLOBAL | ai_action_manager | AddIdle | AddIdle__17ai_action_managerFP9ai_action |
| 0x0033e160 | 136 | GLOBAL | ai_action_manager | NewIdle | NewIdle__17ai_action_managerFP9ai_action |
| 0x0033e1f0 | 88 | GLOBAL | ai_action_manager | AdoptModifiers | AdoptModifiers__17ai_action_managerFUiUi |
| 0x0033e250 | 88 | GLOBAL | ai_action_manager | AdoptIdles | AdoptIdles__17ai_action_managerFUiUi |
| 0x0033e2b0 | 120 | GLOBAL | ai_action_manager | DeriveJob | DeriveJob__17ai_action_managerFP9ai_action |
| 0x0033e330 | 48 | GLOBAL | ai_action_manager | GetJob | GetJob__17ai_action_managerFv |
| 0x0033e360 | 108 | GLOBAL | ai_action_manager | GetBaseJob | GetBaseJob__17ai_action_managerFv |
| 0x0033e3d0 | 864 | GLOBAL | ai_action_manager | NextJob | NextJob__17ai_action_managerFv |
| 0x0033e730 | 132 | GLOBAL | ai_action_manager | NextModifier | NextModifier__17ai_action_managerFv |
| 0x0033e7c0 | 132 | GLOBAL | ai_action_manager | NextIdle | NextIdle__17ai_action_managerFv |
| 0x0033e850 | 108 | GLOBAL | ai_action_manager | hasIdle | hasIdle__17ai_action_managerFP9ai_action |
| 0x0033e8c0 | 276 | GLOBAL | ai_action_manager | hasAction | hasAction__17ai_action_managerFUiPP9ai_actionb |
| 0x0033e9e0 | 84 | GLOBAL | ai_action_manager | GetJob | GetJob__17ai_action_managerFUi |
| 0x0033ea40 | 144 | GLOBAL | ai_action_manager | SignalToChilds | SignalToChilds__17ai_action_managerFUiUi |
| 0x0033ead0 | 44 | GLOBAL | ai_character | Init | Init__12ai_characterFPv |
| 0x0033eb00 | 20 | GLOBAL | ai_character | GetMoralka | GetMoralka__12ai_characterFv |
| 0x0033eb20 | 20 | GLOBAL | ai_character | GetInteligence | GetInteligence__12ai_characterFv |
| 0x0033eb40 | 20 | GLOBAL | ai_character | GetAgresivita | GetAgresivita__12ai_characterFv |
| 0x0033eb60 | 20 | GLOBAL | ai_character | GetZrak | GetZrak__12ai_characterFv |
| 0x0033eb80 | 20 | GLOBAL | ai_character | GetSluch | GetSluch__12ai_characterFv |
| 0x0033eba0 | 2560 | GLOBAL | C_entity | WeaponChoose | WeaponChoose__8C_entityFv |
| 0x0033f5a0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std23allocator<10S_GameItem>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details57compressed_pair_imp<Q23std23allocator<10S_GameItem>,Ui,1>FUi |
| 0x0033f5b0 | 652 | GLOBAL | C_entity | WeaponChoose | WeaponChoose__8C_entityFUi |
| 0x0033f840 | 8 | GLOBAL | C_entity | WeaponGetMaxRange | WeaponGetMaxRange__8C_entityFv |
| 0x0033f850 | 8 | GLOBAL | C_entity | WeaponGetBestRange | WeaponGetBestRange__8C_entityFv |
| 0x0033f860 | 8 | GLOBAL | C_entity | WeaponGetRozptyl | WeaponGetRozptyl__8C_entityFv |
| 0x0033f870 | 8 | GLOBAL | C_entity | WeaponGetReady | WeaponGetReady__8C_entityFv |
| 0x0033f880 | 8 | GLOBAL | C_entity | WeaponIsArmed | WeaponIsArmed__8C_entityFv |
| 0x0033f890 | 136 | GLOBAL | C_entity | WeaponGetInfo | WeaponGetInfo__8C_entityFv |
| 0x0033f920 | 248 | GLOBAL | C_entity | Do_Shoot_Timed | Do_Shoot_Timed__8C_entityFbP8S_vectorP7C_actor |
| 0x0033fa20 | 320 | GLOBAL | C_entity | WeaponTimingSetup | WeaponTimingSetup__8C_entityFUi |
| 0x0033fb60 | 360 | GLOBAL | C_entity | InvalidateActor | InvalidateActor__8C_entityFP7C_actor |
| 0x0033fcd0 | 108 | GLOBAL | C_entity | InvalidateSelf | InvalidateSelf__8C_entityFv |
| 0x0033fd40 | 12 | GLOBAL | C_entity | InvalidatePaths | InvalidatePaths__8C_entityFv |
| 0x0033fd50 | 2072 | GLOBAL | C_entity | CanSafeShoot | CanSafeShoot__8C_entityFP7C_actorbb |
| 0x00340570 | 260 | GLOBAL | C_entity | TraceHuman | TraceHuman__8C_entityFP7C_humanP7ai_blob |
| 0x00340680 | 192 | GLOBAL | C_entity | AddUsingObject | AddUsingObject__8C_entityFv |
| 0x00340740 | 2560 | GLOBAL | C_entity | StateSwitcher | StateSwitcher__8C_entityFv |
| 0x00341140 | 696 | GLOBAL | C_entity | SpawnPolice | SpawnPolice__8C_entityFP7C_humanUiUifUi |
| 0x00341400 | 224 | GLOBAL | C_entity | LockSubstate | LockSubstate__8C_entityFUiUifUib |
| 0x003414e0 | 112 | GLOBAL | C_entity | UnlockSubstate | UnlockSubstate__8C_entityFUi |
| 0x00341550 | 16 | GLOBAL | C_entity | hasLockedState | hasLockedState__8C_entityFv |
| 0x00341560 | 140 | GLOBAL | C_entity | hasLockedSubstate | hasLockedSubstate__8C_entityFUiPUiPf |
| 0x003415f0 | 120 | GLOBAL | C_entity | GetGroupPosition | GetGroupPosition__8C_entityFP7C_actorP8S_vector |
| 0x00341670 | 72 | GLOBAL | C_entity | MemWrite | MemWrite__8C_entityFPPvPvUiUi |
| 0x003416c0 | 72 | GLOBAL | C_entity | MemRead | MemRead__8C_entityFPPvPvUiUi |
| 0x00341710 | 8 | GLOBAL | C_entity | GetVisLogsSize | GetVisLogsSize__8C_entityFv |
| 0x00341720 | 296 | GLOBAL | C_entity | SaveVisLogs | SaveVisLogs__8C_entityFPPv |
| 0x00341850 | 360 | GLOBAL | C_entity | LoadVisLogs | LoadVisLogs__8C_entityFPPv |
| 0x003419c0 | 8 | GLOBAL | C_entity | GetSensorsSize | GetSensorsSize__8C_entityFv |
| 0x003419d0 | 1256 | GLOBAL | C_entity | SaveSensors | SaveSensors__8C_entityFPPv |
| 0x00341ec0 | 1148 | GLOBAL | C_entity | LoadSensors | LoadSensors__8C_entityFPPv |
| 0x00342340 | 80 | GLOBAL | C_entity | GetLogsSize | GetLogsSize__8C_entityFv |
| 0x00342390 | 936 | GLOBAL | C_entity | SaveLogs | SaveLogs__8C_entityFPPv |
| 0x00342740 | 700 | GLOBAL | C_entity | LoadLogs | LoadLogs__8C_entityFPPv |
| 0x00342a00 | 8 | GLOBAL | C_entity | GetCharacterSize | GetCharacterSize__8C_entityFv |
| 0x00342a10 | 256 | GLOBAL | C_entity | SaveCharacter | SaveCharacter__8C_entityFPPv |
| 0x00342b10 | 256 | GLOBAL | C_entity | LoadCharacter | LoadCharacter__8C_entityFPPv |
| 0x00342c10 | 72 | GLOBAL | C_entity | GetStateSize | GetStateSize__8C_entityFv |
| 0x00342c60 | 1012 | GLOBAL | C_entity | SaveState | SaveState__8C_entityFPPv |
| 0x00343060 | 1044 | GLOBAL | C_entity | LoadState | LoadState__8C_entityFPPv |
| 0x00343480 | 300 | GLOBAL | C_entity | GetActionSize | GetActionSize__8C_entityFUi |
| 0x003435b0 | 276 | GLOBAL | C_entity | GetActionManSize | GetActionManSize__8C_entityFv |
| 0x003436d0 | 5836 | GLOBAL | C_entity | SaveActionMan | SaveActionMan__8C_entityFPPv |
| 0x00344da0 | 7568 | GLOBAL | C_entity | LoadActionMan | LoadActionMan__8C_entityFPPv |
| 0x00346b30 | 144 | GLOBAL | C_entity | SaveGameGetSize | SaveGameGetSize__8C_entityFv |
| 0x00346bc0 | 252 | GLOBAL | C_entity | SaveGameSave | SaveGameSave__8C_entityFPPv |
| 0x00346cc0 | 264 | GLOBAL | C_entity | SaveGameLoad | SaveGameLoad__8C_entityFPPv |
| 0x00346dd0 | 1020 | GLOBAL | C_entity | isInEntityFireArea | isInEntityFireArea__8C_entityFP8C_entityP7C_actorP12ai_fire_area |
| 0x003471d0 | 1020 | GLOBAL | C_entity | isInPlayerFireArea | isInPlayerFireArea__8C_entityFP8C_playerP12ai_fire_area |
| 0x003475d0 | 196 | GLOBAL | C_entity | GetFireMetrics | GetFireMetrics__8C_entityFP12ai_fire_area |
| 0x003476a0 | 248 | GLOBAL | C_entity | GetPlayerFireMetrics | GetPlayerFireMetrics__8C_entityFP8C_playerP12ai_fire_area |
| 0x003477a0 | 204 | GLOBAL | C_entity | GetTarget | GetTarget__8C_entityFv |
| 0x00347870 | 900 | GLOBAL | C_entity | BewareCrossFire | BewareCrossFire__8C_entityFUi |
| 0x00347c00 | 84 | GLOBAL | C_entity | isHearedShooting | isHearedShooting__8C_entityFUiUi |
| 0x00347c60 | 320 | GLOBAL | C_entity | canAim | canAim__8C_entityFUi |
| 0x00347da0 | 444 | GLOBAL | ai_fight_modes | Init | Init__14ai_fight_modesFPv |
| 0x00347f60 | 56 | GLOBAL | ai_fight_modes | Destroy | Destroy__14ai_fight_modesFv |
| 0x00347fa0 | 112 | GLOBAL | ai_fight_modes | SetEnemy | SetEnemy__14ai_fight_modesFP7C_actor |
| 0x00348010 | 100 | GLOBAL | ai_fight_modes | Invalidate | Invalidate__14ai_fight_modesFv |
| 0x00348080 | 1896 | GLOBAL | ai_fight_modes | Tick | Tick__14ai_fight_modesFPf |
| 0x003487f0 | 172 | GLOBAL | ai_fight_modes | Hit | Hit__14ai_fight_modesFfb |
| 0x003488a0 | 824 | GLOBAL | ai_fight_modes | Select | Select__14ai_fight_modesFv |
| 0x00348be0 | 36 | GLOBAL | ai_fight_modes | GetHitStat | GetHitStat__14ai_fight_modesFUi |
| 0x00348c10 | 172 | GLOBAL | ai_fight_modes | PrefferMode | PrefferMode__14ai_fight_modesFUifb |
| 0x00348cc0 | 16 | GLOBAL | ai_fight_modes | FinishMode | FinishMode__14ai_fight_modesFv |
| 0x00348cd0 | 40 | GLOBAL | ai_fight_modes | RequestFinishMode | RequestFinishMode__14ai_fight_modesFUi |
| 0x00348d00 | 56 | GLOBAL | ai_fight_modes | ModeMustFinish | ModeMustFinish__14ai_fight_modesFUi |
| 0x00348d40 | 60 | GLOBAL | ai_fight_modes | ForceMode | ForceMode__14ai_fight_modesFUiP7C_actor |
| 0x00348d80 | 480 | GLOBAL | ai_talkman | Tick | Tick__10ai_talkmanFv |
| 0x00348f60 | 596 | GLOBAL | ai_talkman | Say | Say__10ai_talkmanFUiUiUiUiUiUiUi |
| 0x003491c0 | 84 | GLOBAL | ai_talkman | Init | Init__10ai_talkmanFPv |
| 0x00349220 | 56 | GLOBAL | ai_talkman | NewFrame | NewFrame__10ai_talkmanFv |
| 0x00349260 | 204 | GLOBAL | ai_space_coverage | Compute | Compute__17ai_space_coverageFR8S_vector |
| 0x00349330 | 400 | GLOBAL | ai_space_coverage | Tick | Tick__17ai_space_coverageFR8S_vectorR8S_vector |
| 0x003494c0 | 92 | GLOBAL | ai_space_coverage | GetLook | GetLook__17ai_space_coverageFR8S_vector |
| 0x00349520 | 116 | GLOBAL | ai_space_coverage | Preffer | Preffer__17ai_space_coverageFR8S_vectorfb |
| 0x003495a0 | 360 | GLOBAL | C_entity | IsInDanger | IsInDanger__8C_entityFv |
| 0x00349710 | 608 | GLOBAL | C_entity | TickGhost | TickGhost__8C_entityFUi |
| 0x00349970 | 456 | GLOBAL | ai_ghost | Add | Add__8ai_ghostFP7C_actorP18tDynamicCollObjectUif |
| 0x00349b40 | 292 | 13 | gif::vector<14ai_ghost_actor,1> | reserve | reserve__Q23gif26vector<14ai_ghost_actor,1>Fi |
| 0x00349c70 | 156 | GLOBAL | ai_ghost | AddHuman | AddHuman__8ai_ghostFP7C_humanUif |
| 0x00349d10 | 108 | GLOBAL | ai_ghost | Init | Init__8ai_ghostFv |
| 0x00349d80 | 116 | GLOBAL | ai_ghost | Reset | Reset__8ai_ghostFv |
| 0x00349e00 | 280 | GLOBAL | ai_ghost | Invalidate | Invalidate__8ai_ghostFP7C_actorP18tDynamicCollObject |
| 0x00349f20 | 8 | GLOBAL | C_entity | SedniNaPrdelASledujKino | SedniNaPrdelASledujKino__8C_entityFb |
| 0x00349f30 | 8 | GLOBAL | C_entity | VzbudSeUzSouTitulky | VzbudSeUzSouTitulky__8C_entityFv |
| 0x00349f40 | 28 | GLOBAL | C_entity | NeserADejchej | NeserADejchej__8C_entityFv |
| 0x00349f60 | 84 | GLOBAL | C_entity | NeserANedejchej | NeserANedejchej__8C_entityFv |
| 0x00349fc0 | 708 | GLOBAL | C_entity | AddBlockedCon | AddBlockedCon__8C_entityFP11blocked_con |
| 0x0034a290 | 3144 | GLOBAL | C_entity | UpdateAttention | UpdateAttention__8C_entityFv |
| 0x0034aee0 | 268 | 13 | gif::vector<18ai_attention_field,2> | reserve | reserve__Q23gif30vector<18ai_attention_field,2>Fi |
| 0x0034aff0 | 92 | GLOBAL | C_entity | GetAttentionDelay | GetAttentionDelay__8C_entityFP7C_human |
| 0x0034b050 | 240 | GLOBAL | C_entity | SightFilter | SightFilter__8C_entityFv |
| 0x0034b140 | 8 | 13 | C_entity | GetProgram | GetProgram__8C_entityFv |
| 0x0034b150 | 24 | 13 |  | Min<Ui>__FRCUiRCUi | Min<Ui>__FRCUiRCUi |
| 0x0034b170 | 148 | WEAK | std::__vector_imp<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>,0> | erase | erase__Q23std72__vector_imp<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>,0>FP16ai_substate_lock |
| 0x0034b210 | 1172 | WEAK | std::__vector_imp<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>,0> | insert | insert__Q23std72__vector_imp<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>,0>FP16ai_substate_lockUiRC16ai_substate_lock |
| 0x0034b6b0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std29allocator<16ai_substate_lock>,Ui,1> | swap | swap__Q310Metrowerks7details63compressed_pair_imp<Q23std29allocator<16ai_substate_lock>,Ui,1>FRQ310Metrowerks7details63compressed_pair_imp<Q23std29allocator<16ai_substate_lock>,Ui,1> |
| 0x0034b6d0 | 80 | WEAK | std::__vector_deleter<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>> | clear | clear__Q23std74__vector_deleter<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>>Fv |
| 0x0034b720 | 104 | WEAK | std::__vector_deleter<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>> | {dtor} | __dt__Q23std74__vector_deleter<16ai_substate_lock,Q23std29allocator<16ai_substate_lock>>Fv |
| 0x0034b790 | 104 | WEAK | std::__vector_deleter<10S_GameItem,Q23std23allocator<10S_GameItem>> | {dtor} | __dt__Q23std62__vector_deleter<10S_GameItem,Q23std23allocator<10S_GameItem>>Fv |
| 0x0034b800 | 72 | WEAK | std::__vector_deleter<10S_GameItem,Q23std23allocator<10S_GameItem>> | clear | clear__Q23std62__vector_deleter<10S_GameItem,Q23std23allocator<10S_GameItem>>Fv |
| 0x0034b850 | 8 | 13 | C_entity | @3056@CallBack | @3056@CallBack__8C_entityFQ29C_program11E_cb_action |
| 0x0034b860 | 8 | 13 | C_entity | @3056@DecompileCommand | @3056@DecompileCommand__8C_entityFiPc |
| 0x0034b870 | 8 | 13 | C_entity | @3056@Process | @3056@Process__8C_entityFUi |
| 0x0034b880 | 8 | 13 | C_entity | @3056@DecodeInstruction | @3056@DecodeInstruction__8C_entityFPcRi |
| 0x0034b890 | 8 | 13 | C_entity | @3056@DecodeParams | @3056@DecodeParams__8C_entityFPcRQ29C_program13S_instruction |
| 0x0034b8a0 | 8 | 13 | C_entity | @3056@SaveGameLoad | @3056@SaveGameLoad__8C_entityFPPv |
| 0x0034b8b0 | 8 | 13 | C_entity | @3056@SaveGameSave | @3056@SaveGameSave__8C_entityFPPv |
| 0x0034b8c0 | 8 | 13 | C_entity | @3056@SaveGameGetSize | @3056@SaveGameGetSize__8C_entityFv |
| 0x0034b8d0 | 8 | 13 |  | @3056@__dt__8C_entityFv | @3056@__dt__8C_entityFv |
| 0x0034b8e0 | 160 | GLOBAL | ai_group | Add | Add__8ai_groupFP8C_entityUi |
| 0x0034b980 | 124 | GLOBAL | ai_group | Del | Del__8ai_groupFP8C_entity |
| 0x0034ba00 | 272 | GLOBAL | ai_group | GetSniff | GetSniff__8ai_groupFP8C_entityP7C_actor |
| 0x0034bb10 | 156 | GLOBAL | ai_group | UnderAttack | UnderAttack__8ai_groupFP8C_entityP7C_actorf |
| 0x0034bbb0 | 436 | GLOBAL | ai_group | ProcessEvents | ProcessEvents__8ai_groupFUi |
| 0x0034bd70 | 104 | GLOBAL | ai_group | GetCarFactor | GetCarFactor__8ai_groupFP5C_car |
| 0x0034bde0 | 1212 | GLOBAL | ai_group | GetCarClosestAndUsable | GetCarClosestAndUsable__8ai_groupFP8S_vectorP8S_vectorPUi |
| 0x0034c2a0 | 600 | GLOBAL | ai_groups | NewGroup | NewGroup__9ai_groupsFv |
| 0x0034c500 | 304 | 13 | ai_group | {ctor} | __ct__8ai_groupFRC8ai_group |
| 0x0034c630 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14ai_group_event>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14ai_group_event>,Ui,1>FRCQ23std27allocator<14ai_group_event>Ui |
| 0x0034c640 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14ai_group_event>,Ui,1> | first | first__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14ai_group_event>,Ui,1>CFv |
| 0x0034c650 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15ai_group_member>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15ai_group_member>,Ui,1>FRCQ23std28allocator<15ai_group_member>Ui |
| 0x0034c660 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15ai_group_member>,Ui,1> | first | first__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15ai_group_member>,Ui,1>CFv |
| 0x0034c670 | 784 | 13 | gif::vector<8ai_group,2> | reserve | reserve__Q23gif19vector<8ai_group,2>Fi |
| 0x0034c980 | 28 | 13 | std::__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0> | end | end__Q23std68__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0>CFv |
| 0x0034c9a0 | 8 | 13 | std::__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0> | begin | begin__Q23std68__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0>CFv |
| 0x0034c9b0 | 20 | 13 | std::__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>> | {ctor} | __ct__Q23std70__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>>FRCQ23std27allocator<14ai_group_event> |
| 0x0034c9d0 | 8 | 13 | std::__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>> | alloc | alloc__Q23std70__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>>CFv |
| 0x0034c9e0 | 20 | 13 | std::__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0> | end | end__Q23std70__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0>CFv |
| 0x0034ca00 | 8 | 13 | std::__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0> | begin | begin__Q23std70__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0>CFv |
| 0x0034ca10 | 20 | 13 | std::__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>> | {ctor} | __ct__Q23std72__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>>FRCQ23std28allocator<15ai_group_member> |
| 0x0034ca30 | 8 | 13 | std::__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>> | alloc | alloc__Q23std72__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>>CFv |
| 0x0034ca40 | 216 | 13 | ai_group | {dtor} | __dt__8ai_groupFv |
| 0x0034cb20 | 84 | 13 | std::__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0> | {dtor} | __dt__Q23std68__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0>Fv |
| 0x0034cb80 | 20 | 13 | std::__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>> | {ctor} | __ct__Q23std70__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>>Fv |
| 0x0034cba0 | 84 | 13 | std::__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0> | {dtor} | __dt__Q23std70__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0>Fv |
| 0x0034cc00 | 20 | 13 | std::__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>> | {ctor} | __ct__Q23std72__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>>Fv |
| 0x0034cc20 | 128 | GLOBAL | ai_groups | GetGroup | GetGroup__9ai_groupsFUi |
| 0x0034cca0 | 192 | GLOBAL | ai_groups | Init | Init__9ai_groupsFv |
| 0x0034cd60 | 364 | GLOBAL | ai_groups | Destroy | Destroy__9ai_groupsFv |
| 0x0034ced0 | 620 | GLOBAL | ai_groups | Tick | Tick__9ai_groupsFv |
| 0x0034d140 | 200 | GLOBAL | ai_groups | GetSize | GetSize__9ai_groupsFv |
| 0x0034d210 | 1072 | GLOBAL | ai_groups | Save | Save__9ai_groupsFPPv |
| 0x0034d640 | 1824 | GLOBAL | ai_groups | Load | Load__9ai_groupsFPPv |
| 0x0034dd60 | 260 | 13 | gif::vector<P7C_actor,2> | reserve | reserve__Q23gif19vector<P7C_actor,2>Fi |
| 0x0034de70 | 832 | GLOBAL | ai_groups | InvalidateActor | InvalidateActor__9ai_groupsFP7C_actor |
| 0x0034e1b0 | 80 | 13 | std::__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0> | operator= | __as__Q23std68__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0>FRCQ23std68__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0> |
| 0x0034e200 | 72 | 13 | std::__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0> | operator= | __as__Q23std70__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0>FRCQ23std70__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0> |
| 0x0034e250 | 92 | GLOBAL | ai_group | canReceive | canReceive__8ai_groupFP8C_entity |
| 0x0034e2b0 | 92 | GLOBAL | ai_group | canCast | canCast__8ai_groupFP8C_entity |
| 0x0034e310 | 144 | GLOBAL | ai_groups | hasCarGroup | hasCarGroup__9ai_groupsFUi |
| 0x0034e3a0 | 72 | GLOBAL | ai_groups | MemWrite | MemWrite__9ai_groupsFPPvPvUiUi |
| 0x0034e3f0 | 72 | GLOBAL | ai_groups | MemRead | MemRead__9ai_groupsFPPvPvUiUi |
| 0x0034e440 | 460 | GLOBAL | ai_follow_manager | DelFollower | DelFollower__17ai_follow_managerFP7C_actor |
| 0x0034e610 | 540 | 13 | gif::vector<P7C_actor,8> | operator= | __as__Q23gif19vector<P7C_actor,8>FRCQ23gif19vector<P7C_actor,8> |
| 0x0034e830 | 1024 | GLOBAL | ai_follow_manager | AddFollower | AddFollower__17ai_follow_managerFP7C_actorP7C_actor |
| 0x0034ec30 | 204 | 13 | ai_follow_group | {ctor} | __ct__15ai_follow_groupFRC15ai_follow_group |
| 0x0034ed00 | 684 | 13 | gif::vector<P7C_actor,8> | push_back | push_back__Q23gif19vector<P7C_actor,8>FRCP7C_actor |
| 0x0034efb0 | 364 | 13 | gif::vector<15ai_follow_group,1> | reserve | reserve__Q23gif27vector<15ai_follow_group,1>Fi |
| 0x0034f120 | 580 | 13 | gif::vector<P7C_actor,8> | {ctor} | __ct__Q23gif19vector<P7C_actor,8>FRCQ23gif19vector<P7C_actor,8> |
| 0x0034f370 | 1120 | GLOBAL | ai_follow_manager | AddFollower | AddFollower__17ai_follow_managerFP7C_actorP7C_actorR8S_vector |
| 0x0034f7d0 | 2360 | GLOBAL | ai_follow_manager | Update | Update__17ai_follow_managerFv |
| 0x00350110 | 120 | GLOBAL | ai_follow_manager | Init | Init__17ai_follow_managerFv |
| 0x00350190 | 280 | GLOBAL | ai_follow_manager | Destroy | Destroy__17ai_follow_managerFv |
| 0x003502b0 | 436 | GLOBAL | ai_follow_manager | CanFolloshipFollow | CanFolloshipFollow__17ai_follow_managerFP7C_actor |
| 0x00350470 | 568 | GLOBAL | ai_follow_manager | GetCarSeat | GetCarSeat__17ai_follow_managerFP7C_actorP7C_actor |
| 0x003506b0 | 444 | GLOBAL | ai_follow_manager | Invalidate | Invalidate__17ai_follow_managerFP7C_actor |
| 0x00350870 | 488 | GLOBAL | ai_follow_manager | Event | Event__17ai_follow_managerFUiP7C_actor |
| 0x00350a60 | 8 | GLOBAL | ai_follow_manager | mustEnter | mustEnter__17ai_follow_managerFP7C_actor |
| 0x00350a70 | 324 | GLOBAL | ai_follow_manager | mustLeave | mustLeave__17ai_follow_managerFP7C_actor |
| 0x00350bc0 | 72 | GLOBAL | ai_follow_manager | MemWrite | MemWrite__17ai_follow_managerFPPvPvUiUi |
| 0x00350c10 | 72 | GLOBAL | ai_follow_manager | MemRead | MemRead__17ai_follow_managerFPPvPvUiUi |
| 0x00350c60 | 120 | GLOBAL | ai_follow_manager | GetSize | GetSize__17ai_follow_managerFv |
| 0x00350ce0 | 604 | GLOBAL | ai_follow_manager | Save | Save__17ai_follow_managerFPPv |
| 0x00350f40 | 1304 | GLOBAL | ai_follow_manager | Load | Load__17ai_follow_managerFPPv |
| 0x00351460 | 888 | GLOBAL | ai_fight_manager | JoinParty | JoinParty__16ai_fight_managerFP8C_entityP7C_actor |
| 0x003517e0 | 236 | 13 | ai_fight_followship | {ctor} | __ct__19ai_fight_followshipFRC19ai_fight_followship |
| 0x003518d0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15ai_fight_member>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15ai_fight_member>,Ui,1>FRCQ23std28allocator<15ai_fight_member>Ui |
| 0x003518e0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15ai_fight_member>,Ui,1> | first | first__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15ai_fight_member>,Ui,1>CFv |
| 0x003518f0 | 372 | 13 | gif::vector<19ai_fight_followship,2> | reserve | reserve__Q23gif31vector<19ai_fight_followship,2>Fi |
| 0x00351a70 | 92 | 13 | std::vector<15ai_fight_member,Q23std28allocator<15ai_fight_member>> | {dtor} | __dt__Q23std62vector<15ai_fight_member,Q23std28allocator<15ai_fight_member>>Fv |
| 0x00351ad0 | 84 | 13 | std::__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0> | {dtor} | __dt__Q23std70__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0>Fv |
| 0x00351b30 | 20 | 13 | std::__vector_deleter<15ai_fight_member,Q23std28allocator<15ai_fight_member>> | {ctor} | __ct__Q23std72__vector_deleter<15ai_fight_member,Q23std28allocator<15ai_fight_member>>Fv |
| 0x00351b50 | 788 | GLOBAL | ai_fight_manager | LeaveParty | LeaveParty__16ai_fight_managerFP8C_entityP7C_actor |
| 0x00351e70 | 200 | 13 | ai_fight_followship | operator= | __as__19ai_fight_followshipFRC19ai_fight_followship |
| 0x00351f40 | 328 | GLOBAL | ai_fight_followship | SetCommander | SetCommander__19ai_fight_followshipFv |
| 0x00352090 | 480 | GLOBAL | ai_fight_followship | CountFreeSeats | CountFreeSeats__19ai_fight_followshipFP5C_car |
| 0x00352270 | 880 | GLOBAL | ai_fight_followship | TakeCar | TakeCar__19ai_fight_followshipFP5C_car |
| 0x003525e0 | 152 | GLOBAL | ai_fight_manager | NewCarParty | NewCarParty__16ai_fight_managerFP5C_carP7C_actor |
| 0x00352680 | 364 | GLOBAL | ai_fight_followship | AddCarMember | AddCarMember__19ai_fight_followshipFP8C_entity |
| 0x003527f0 | 432 | GLOBAL | ai_fight_followship | ComputePosition | ComputePosition__19ai_fight_followshipFv |
| 0x003529a0 | 8 | 13 | std::__vector_deleter<15ai_fight_member,Q23std28allocator<15ai_fight_member>> | data | data__Q23std72__vector_deleter<15ai_fight_member,Q23std28allocator<15ai_fight_member>>Fv |
| 0x003529b0 | 792 | GLOBAL | ai_fight_followship | mustCarHunt | mustCarHunt__19ai_fight_followshipFv |
| 0x00352cd0 | 528 | GLOBAL | ai_fight_manager | DisbandCarParty | DisbandCarParty__16ai_fight_managerFP19ai_fight_followshipb |
| 0x00352ee0 | 128 | GLOBAL | ai_fight_manager | AdjustFormTime | AdjustFormTime__16ai_fight_managerFP7C_actor |
| 0x00352f60 | 272 | GLOBAL | ai_fight_manager | GetParty | GetParty__16ai_fight_managerFP8C_entityP7C_actorPUi |
| 0x00353070 | 120 | GLOBAL | ai_fight_manager | Init | Init__16ai_fight_managerFv |
| 0x003530f0 | 236 | GLOBAL | ai_fight_manager | Destroy | Destroy__16ai_fight_managerFv |
| 0x003531e0 | 6160 | GLOBAL | ai_fight_manager | Tick | Tick__16ai_fight_managerFv |
| 0x003549f0 | 596 | GLOBAL | ai_fight_manager | Invalidate | Invalidate__16ai_fight_managerFP7C_actor |
| 0x00354c50 | 268 | 13 | std::__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0> | init<PC15ai_group_member> | init<PC15ai_group_member>__Q23std70__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0>FPC15ai_group_memberPC15ai_group_memberQ23std20forward_iterator_tag |
| 0x00354d60 | 568 | 13 | std::__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0> | do_assign<PC15ai_group_member> | do_assign<PC15ai_group_member>__Q23std70__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0>FPC15ai_group_memberPC15ai_group_memberQ23std20forward_iterator_tag |
| 0x00354fa0 | 332 | 13 | std::__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0> | init<PC14ai_group_event> | init<PC14ai_group_event>__Q23std68__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0>FPC14ai_group_eventPC14ai_group_eventQ23std20forward_iterator_tag |
| 0x003550f0 | 680 | 13 | std::__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0> | do_assign<PC14ai_group_event> | do_assign<PC14ai_group_event>__Q23std68__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0>FPC14ai_group_eventPC14ai_group_eventQ23std20forward_iterator_tag |
| 0x003553a0 | 332 | 13 | std::__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0> | init<PC15ai_fight_member> | init<PC15ai_fight_member>__Q23std70__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0>FPC15ai_fight_memberPC15ai_fight_memberQ23std20forward_iterator_tag |
| 0x003554f0 | 680 | 13 | std::__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0> | do_assign<PC15ai_fight_member> | do_assign<PC15ai_fight_member>__Q23std70__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0>FPC15ai_fight_memberPC15ai_fight_memberQ23std20forward_iterator_tag |
| 0x003557a0 | 116 | WEAK | std::__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0> | erase | erase__Q23std70__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0>FP15ai_group_member |
| 0x00355820 | 1052 | WEAK | std::__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0> | insert | insert__Q23std70__vector_imp<15ai_group_member,Q23std28allocator<15ai_group_member>,0>FP15ai_group_memberUiRC15ai_group_member |
| 0x00355c40 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15ai_group_member>,Ui,1> | swap | swap__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15ai_group_member>,Ui,1>FRQ310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15ai_group_member>,Ui,1> |
| 0x00355c60 | 164 | WEAK | std::__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0> | erase | erase__Q23std68__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0>FP14ai_group_event |
| 0x00355d10 | 1300 | WEAK | std::__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0> | insert | insert__Q23std68__vector_imp<14ai_group_event,Q23std27allocator<14ai_group_event>,0>FP14ai_group_eventUiRC14ai_group_event |
| 0x00356230 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14ai_group_event>,Ui,1> | swap | swap__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14ai_group_event>,Ui,1>FRQ310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14ai_group_event>,Ui,1> |
| 0x00356250 | 164 | WEAK | std::__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0> | erase | erase__Q23std70__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0>FP15ai_fight_member |
| 0x00356300 | 1300 | WEAK | std::__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0> | insert | insert__Q23std70__vector_imp<15ai_fight_member,Q23std28allocator<15ai_fight_member>,0>FP15ai_fight_memberUiRC15ai_fight_member |
| 0x00356820 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15ai_fight_member>,Ui,1> | swap | swap__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15ai_fight_member>,Ui,1>FRQ310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15ai_fight_member>,Ui,1> |
| 0x00356840 | 72 | WEAK | std::__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>> | clear | clear__Q23std72__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>>Fv |
| 0x00356890 | 104 | WEAK | std::__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>> | {dtor} | __dt__Q23std72__vector_deleter<15ai_group_member,Q23std28allocator<15ai_group_member>>Fv |
| 0x00356900 | 80 | WEAK | std::__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>> | clear | clear__Q23std70__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>>Fv |
| 0x00356950 | 104 | WEAK | std::__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>> | {dtor} | __dt__Q23std70__vector_deleter<14ai_group_event,Q23std27allocator<14ai_group_event>>Fv |
| 0x003569c0 | 80 | WEAK | std::__vector_deleter<15ai_fight_member,Q23std28allocator<15ai_fight_member>> | clear | clear__Q23std72__vector_deleter<15ai_fight_member,Q23std28allocator<15ai_fight_member>>Fv |
| 0x00356a10 | 104 | WEAK | std::__vector_deleter<15ai_fight_member,Q23std28allocator<15ai_fight_member>> | {dtor} | __dt__Q23std72__vector_deleter<15ai_fight_member,Q23std28allocator<15ai_fight_member>>Fv |
| 0x00356a80 | 1280 | GLOBAL | ai_check_builder | TestSpace8RExt | TestSpace8RExt__16ai_check_builderFR8S_vectorfPf |
| 0x00356f80 | 116 | GLOBAL | ai_check_builder | RotateVect | RotateVect__16ai_check_builderFR8S_vectorf |
| 0x00357000 | 300 | GLOBAL | ai_check_builder | GetOrientation | GetOrientation__16ai_check_builderFR8S_vector |
| 0x00357130 | 444 | GLOBAL |  | GetBoundPoints2D__FRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame | GetBoundPoints2D__FRQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>P9I3D_frame |
| 0x003572f0 | 304 | GLOBAL |  | fxForwardShortest__FUsUsUc | fxForwardShortest__FUsUsUc |
| 0x00357420 | 256 | GLOBAL |  | fxRunawayEndFuncV1__FUs | fxRunawayEndFuncV1__FUs |
| 0x00357520 | 596 | GLOBAL |  | fxRunawayFuncV1__FUsUsUc | fxRunawayFuncV1__FUsUsUc |
| 0x00357780 | 72 | GLOBAL |  | fxForward2EndFuncV1__FUs | fxForward2EndFuncV1__FUs |
| 0x003577d0 | 1632 | GLOBAL |  | fxForward2FuncV1__FUsUsUc | fxForward2FuncV1__FUsUsUc |
| 0x00357e30 | 72 | GLOBAL | ai_check_desc | UNPACKDATA | UNPACKDATA__13ai_check_descFUi |
| 0x00357e80 | 212 | GLOBAL | ai_check_desc | UNPACKRANGES | UNPACKRANGES__13ai_check_descFUi |
| 0x00357f60 | 1744 | GLOBAL | ai_wayfind | GetPathNode | GetPathNode__10ai_wayfindFP10C_web_nodeP10C_web_nodeP10C_web_nodeP12ai_path_node |
| 0x00358630 | 588 | GLOBAL | ai_wayfind | GetPathNode | GetPathNode__10ai_wayfindFP10C_web_nodeRC8S_vectorRC8S_vectorP12ai_path_node |
| 0x00358880 | 72 | 13 | ai_path_node | {ctor} | __ct__12ai_path_nodeFRC12ai_path_node |
| 0x003588d0 | 388 | 13 | gif::vector<12ai_path_node,4> | reserve | reserve__Q23gif24vector<12ai_path_node,4>Fi |
| 0x00358a60 | 8548 | GLOBAL | ai_wayfind | GetOptimalPath | GetOptimalPath__10ai_wayfindFv |
| 0x0035abd0 | 1608 | GLOBAL | ai_wayfind | FindStartCheckpoint | FindStartCheckpoint__10ai_wayfindFR8S_vectorUib |
| 0x0035b220 | 1208 | GLOBAL | ai_wayfind | FindEndCheckpoint | FindEndCheckpoint__10ai_wayfindFR8S_vectorUib |
| 0x0035b6e0 | 88 | GLOBAL | ai_wayfind | GetPosition | GetPosition__10ai_wayfindFR8S_vectorPUiPUiUi |
| 0x0035b740 | 668 | GLOBAL | ai_wayfind | FindWaySHORTEST | FindWaySHORTEST__10ai_wayfindFR8S_vectorR8S_vectorUib |
| 0x0035b9e0 | 1680 | GLOBAL | ai_wayfind | FindWayCORNERNEAR | FindWayCORNERNEAR__10ai_wayfindFR8S_vectorR8S_vectorUiP19ai_corner_hide_infob |
| 0x0035c070 | 1104 | GLOBAL | ai_wayfind | FindWayRUNAWAY_V1 | FindWayRUNAWAY_V1__10ai_wayfindFR8S_vectorR8S_vectorfUib |
| 0x0035c4c0 | 1460 | GLOBAL | ai_wayfind | FindWayFORWARD_V2 | FindWayFORWARD_V2__10ai_wayfindFR8S_vectorR8S_vectorfUib |
| 0x0035ca80 | 1728 | GLOBAL | ai_wayfind | FindWayFORWARD_V3 | FindWayFORWARD_V3__10ai_wayfindFR8S_vectorR8S_vectorP7C_humanP7C_humanfUib |
| 0x0035d140 | 892 | GLOBAL | ai_wayfind | FindWaySEARCH | FindWaySEARCH__10ai_wayfindFP15ai_check_searchUiUib |
| 0x0035d4c0 | 1044 | GLOBAL | ai_wayfind | hasCornerVisHide | hasCornerVisHide__10ai_wayfindFP13ai_seg_cornerR8S_vectorR8S_vectorR8S_vectorPUibf |
| 0x0035d8e0 | 264 | GLOBAL | ai_path | Destroy | Destroy__7ai_pathFv |
| 0x0035d9f0 | 8348 | GLOBAL | ai_path | GetSpecNodeA | GetSpecNodeA__7ai_pathFR8S_vectorR8S_vectorR7S_planeP7C_actorRQ23gif31vector<P18g_collision_header,8>UiP7C_actor |
| 0x0035fa90 | 292 | 13 | gif::vector<16ai_path_specnode,2> | reserve | reserve__Q23gif28vector<16ai_path_specnode,2>Fi |
| 0x0035fbc0 | 284 | 13 | gif::vector<16ai_actor_evading,1> | reserve | reserve__Q23gif28vector<16ai_actor_evading,1>Fi |
| 0x0035fce0 | 8 | 13 | std::__vector_deleter<9S_vector2,Q23std21allocator<9S_vector2>> | data | data__Q23std58__vector_deleter<9S_vector2,Q23std21allocator<9S_vector2>>Fv |
| 0x0035fcf0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std21allocator<9S_vector2>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details55compressed_pair_imp<Q23std21allocator<9S_vector2>,Ui,1>FUi |
| 0x0035fd00 | 7564 | GLOBAL | ai_path | GetSpecNodeB | GetSpecNodeB__7ai_pathFR8S_vectorR8S_vectorR7S_planeP7C_actorUiP7C_actor |
| 0x00361a90 | 7844 | GLOBAL | ai_path | GetSpecNodeC | GetSpecNodeC__7ai_pathFR8S_vectorR8S_vectorP7C_actorUiP7C_actor |
| 0x00363940 | 2736 | GLOBAL | ai_path | UpdateSpec | UpdateSpec__7ai_pathFR8S_vectorbP7C_actor |
| 0x003643f0 | 936 | GLOBAL | ai_path | GetNextNormalWaypoint | GetNextNormalWaypoint__7ai_pathFR8S_vectorb |
| 0x003647a0 | 864 | GLOBAL | ai_path | GetNormalWaypoint | GetNormalWaypoint__7ai_pathFR8S_vector |
| 0x00364b00 | 200 | GLOBAL | ai_path | GetNextSpecWaypoint | GetNextSpecWaypoint__7ai_pathFR8S_vector |
| 0x00364bd0 | 108 | GLOBAL | ai_path | isLastNormalWaypoint | isLastNormalWaypoint__7ai_pathFv |
| 0x00364c40 | 1048 | GLOBAL | ai_path | isInConnection | isInConnection__7ai_pathFRC8S_vectorRC8S_vectorRC8S_vectorff |
| 0x00365060 | 1124 | GLOBAL | ai_path | isInConnection | isInConnection__7ai_pathFRC8S_vectorRC8S_vectorRC9S_vector2ff |
| 0x003654d0 | 64 | GLOBAL | ai_path | isInCheckpoint | isInCheckpoint__7ai_pathFRC8S_vectorRC9S_vector2f |
| 0x00365510 | 1284 | GLOBAL | ai_path | Tick | Tick__7ai_pathFUiP7C_humanbb |
| 0x00365a20 | 1636 | GLOBAL | ai_path | Update | Update__7ai_pathFP7C_actor |
| 0x00366090 | 104 | GLOBAL | ai_path | GetDestination | GetDestination__7ai_pathFR8S_vector |
| 0x00366100 | 24 | GLOBAL | ai_path | DelayTest | DelayTest__7ai_pathFv |
| 0x00366120 | 8 | GLOBAL | ai_path | mustEvade | mustEvade__7ai_pathFP7C_actorR9S_vector2R9S_vector2 |
| 0x00366130 | 560 | GLOBAL | ai_path | GetSegments | GetSegments__7ai_pathFUib |
| 0x00366360 | 996 | GLOBAL | ai_wayfind | FindCORNERNEAR | FindCORNERNEAR__10ai_wayfindFR8S_vectorR8S_vectorP19ai_corner_hide_info |
| 0x00366750 | 1496 | GLOBAL | ai_wayfind | WeightCheckpoints | WeightCheckpoints__10ai_wayfindFP15ai_check_searchPQ23std32vector<Ui,Q23std13allocator<Ui>>UiUiUiUi |
| 0x00366d30 | 392 | GLOBAL | ai_path | areOverlaped | areOverlaped__7ai_pathFR8S_vectorR8S_vectorR8S_vectorffff |
| 0x00366ec0 | 168 | GLOBAL | ai_wayfind | Compute2DIntersection | Compute2DIntersection__10ai_wayfindFP9S_vector2P9S_vector2P9S_vector2P9S_vector2P9S_vector2 |
| 0x00366f70 | 3428 | GLOBAL | ai_wayfind | GetLR | GetLR__10ai_wayfindFP8S_vectorP8S_vectorP8S_vectorP12ai_path_nodefff |
| 0x00367ce0 | 400 | GLOBAL | ai_wayfind | ComputeNearestPoint | ComputeNearestPoint__10ai_wayfindFP8S_vectorP8S_vectorP8S_vectorP8S_vectorff |
| 0x00367e70 | 260 | GLOBAL | ai_wayfind | isInBound | isInBound__10ai_wayfindFPQ23std48vector<9S_vector2,Q23std21allocator<9S_vector2>>R8S_vector |
| 0x00367f80 | 80 | GLOBAL | ai_path | ForceRight | ForceRight__7ai_pathFv |
| 0x00367fd0 | 728 | GLOBAL | ai_path | isInNextConn | isInNextConn__7ai_pathFR8S_vector |
| 0x003682b0 | 828 | GLOBAL | ai_path | TestDetailLines | TestDetailLines__7ai_pathFP8S_vectorP8S_vectorP8S_vectorPP18tDynamicCollObject |
| 0x003685f0 | 748 | GLOBAL | ai_path | isTurningNormalNode | isTurningNormalNode__7ai_pathFR8S_vector |
| 0x003688e0 | 92 | GLOBAL | ai_path | operator new | __nw__7ai_pathFUi |
| 0x00368940 | 16 | GLOBAL | ai_path | operator delete | __dl__7ai_pathFPv |
| 0x00368950 | 20 | GLOBAL | ai_manager | ResetCounting | ResetCounting__10ai_managerFv |
| 0x00368970 | 188 | GLOBAL | ai_sight_manager | Init | Init__16ai_sight_managerFv |
| 0x00368a30 | 624 | GLOBAL | ai_sight_manager | Add | Add__16ai_sight_managerFP8C_entityb |
| 0x00368ca0 | 260 | 13 | gif::vector<P8C_entity,8> | reserve | reserve__Q23gif20vector<P8C_entity,8>Fi |
| 0x00368db0 | 368 | GLOBAL | ai_sight_manager | Del | Del__16ai_sight_managerFP8C_entity |
| 0x00368f20 | 108 | GLOBAL | ai_sight_manager | Next | Next__16ai_sight_managerFv |
| 0x00368f90 | 108 | GLOBAL | ai_sight_manager | NextCorpses | NextCorpses__16ai_sight_managerFv |
| 0x00369000 | 248 | GLOBAL | ai_sight_manager | Tick | Tick__16ai_sight_managerFv |
| 0x00369100 | 248 | GLOBAL | ai_sight_manager | TickCorpses | TickCorpses__16ai_sight_managerFv |
| 0x00369200 | 312 | GLOBAL | ai_panic_manager | Add | Add__16ai_panic_managerFP8C_entity |
| 0x00369340 | 128 | GLOBAL | ai_panic_manager | Del | Del__16ai_panic_managerFP8C_entity |
| 0x003693c0 | 116 | GLOBAL | ai_panic_manager | Init | Init__16ai_panic_managerFUi |
| 0x00369440 | 480 | GLOBAL | ai_panic_manager | Tick | Tick__16ai_panic_managerFv |
| 0x00369620 | 232 | GLOBAL | ai_panic_manager | FreezeAll | FreezeAll__16ai_panic_managerFUi |
| 0x00369710 | 240 | GLOBAL | ai_framerater | Tick | Tick__13ai_frameraterFUi |
| 0x00369800 | 160 | GLOBAL | ai_framerater | GetFPS | GetFPS__13ai_frameraterFv |
| 0x003698a0 | 12 | GLOBAL | ai_framerater | Init | Init__13ai_frameraterFv |
| 0x003698b0 | 172 | GLOBAL | ai_spageman | Init | Init__11ai_spagemanFUiUiUi |
| 0x00369960 | 228 | GLOBAL | ai_spageman | Destroy | Destroy__11ai_spagemanFv |
| 0x00369a50 | 512 | GLOBAL | ai_spageman | Alloc | Alloc__11ai_spagemanFv |
| 0x00369c50 | 260 | 13 | gif::vector<P8ai_spage,8> | reserve | reserve__Q23gif20vector<P8ai_spage,8>Fi |
| 0x00369d60 | 404 | GLOBAL | ai_spageman | Free | Free__11ai_spagemanFPv |
| 0x00369f00 | 400 | GLOBAL | ai_spageman | Compactify | Compactify__11ai_spagemanFPPv |
| 0x0036a090 | 344 | GLOBAL | ai_spageman | FreeUnused | FreeUnused__11ai_spagemanFUi |
| 0x0036a1f0 | 140 | 13 | ai_spageman | ComputeUsageRatio | ComputeUsageRatio__11ai_spagemanFv |
| 0x0036a280 | 164 | GLOBAL | ai_spage | Alloc | Alloc__8ai_spageFv |
| 0x0036a330 | 124 | GLOBAL | ai_spage | Free | Free__8ai_spageFPv |
| 0x0036a3b0 | 140 | GLOBAL | ai_spage | Init | Init__8ai_spageFUiUi |
| 0x0036a440 | 88 | GLOBAL | ai_spage | Destroy | Destroy__8ai_spageFv |
| 0x0036a4a0 | 216 | GLOBAL | ai_spage | GetSlot | GetSlot__8ai_spageFUib |
| 0x0036a580 | 96 | GLOBAL | ai_spage | isValid | isValid__8ai_spageFPv |
| 0x0036a5e0 | 268 | GLOBAL | ai_manager | Init | Init__10ai_managerFv |
| 0x0036a6f0 | 104 | GLOBAL | ai_manager | Tick | Tick__10ai_managerFUi |
| 0x0036a760 | 188 | GLOBAL | ai_manager | Destroy | Destroy__10ai_managerFv |
| 0x0036a820 | 292 | GLOBAL | ai_unlock_manager | Unlock | Unlock__17ai_unlock_managerFP7C_actorUi |
| 0x0036a950 | 268 | 13 | gif::vector<14ai_unlock_item,8> | reserve | reserve__Q23gif26vector<14ai_unlock_item,8>Fi |
| 0x0036aa60 | 144 | GLOBAL | ai_unlock_manager | Invalidate | Invalidate__17ai_unlock_managerFP7C_actor |
| 0x0036aaf0 | 340 | GLOBAL | ai_unlock_manager | Tick | Tick__17ai_unlock_managerFUi |
| 0x0036ac50 | 888 | 13 | ai_manager | {dtor} | __dt__10ai_managerFv |
| 0x0036afd0 | 84 | 13 | std::__vector_imp<12ai_anim_item,Q23std25allocator<12ai_anim_item>,0> | {dtor} | __dt__Q23std64__vector_imp<12ai_anim_item,Q23std25allocator<12ai_anim_item>,0>Fv |
| 0x0036b030 | 20 | 13 | std::__vector_imp<12ai_anim_item,Q23std25allocator<12ai_anim_item>,0> | {ctor} | __ct__Q23std64__vector_imp<12ai_anim_item,Q23std25allocator<12ai_anim_item>,0>Fv |
| 0x0036b050 | 8 | 13 | Metrowerks::compile_assert<1> | {ctor} | __ct__Q210Metrowerks17compile_assert<1>Fv |
| 0x0036b060 | 104 | WEAK | std::__vector_deleter<12ai_anim_item,Q23std25allocator<12ai_anim_item>> | {dtor} | __dt__Q23std66__vector_deleter<12ai_anim_item,Q23std25allocator<12ai_anim_item>>Fv |
| 0x0036b0d0 | 72 | WEAK | std::__vector_deleter<12ai_anim_item,Q23std25allocator<12ai_anim_item>> | clear | clear__Q23std66__vector_deleter<12ai_anim_item,Q23std25allocator<12ai_anim_item>>Fv |
| 0x0036b120 | 368 | GLOBAL | ai_police_action | HosiKoncime | HosiKoncime__16ai_police_actionFP7C_actorb |
| 0x0036b290 | 156 | GLOBAL | ai_police_action | EventVsem | EventVsem__16ai_police_actionFP7C_actorUiPv |
| 0x0036b330 | 12128 | GLOBAL | ai_police_action | Tick | Tick__16ai_police_actionFUi |
| 0x0036e290 | 80 | GLOBAL | ai_police_action | KolikatejSemSefe | KolikatejSemSefe__16ai_police_actionFP7C_actor |
| 0x0036e2e0 | 1752 | GLOBAL | ai_police_action | PanoveCekamVasNaWCSesynchronizujemSiCas | PanoveCekamVasNaWCSesynchronizujemSiCas__16ai_police_actionFUi |
| 0x0036e9c0 | 832 | GLOBAL | ai_police_action | TakZnovuDoToho | TakZnovuDoToho__16ai_police_actionFv |
| 0x0036ed00 | 232 | GLOBAL | ai_police_action | ZbavSeAutaSmrdi | ZbavSeAutaSmrdi__16ai_police_actionFb |
| 0x0036edf0 | 156 | GLOBAL | ai_police_action | ProhodteSiHodnosti | ProhodteSiHodnosti__16ai_police_actionFv |
| 0x0036ee90 | 160 | GLOBAL | ai_police_nastenka | ZacatekSmeny | ZacatekSmeny__18ai_police_nastenkaFv |
| 0x0036ef30 | 16 | GLOBAL | ai_police_nastenka | JaChcuCislo | JaChcuCislo__18ai_police_nastenkaFv |
| 0x0036ef40 | 148 | GLOBAL | ai_police_nastenka | CoVlastneRobimPlukovniku | CoVlastneRobimPlukovniku__18ai_police_nastenkaFP7C_actor |
| 0x0036efe0 | 92 | GLOBAL | ai_police_nastenka | MamCisloDejAkci | MamCisloDejAkci__18ai_police_nastenkaFUi |
| 0x0036f040 | 3352 | GLOBAL | ai_police_nastenka | MamProVasAkciHosi | MamProVasAkciHosi__18ai_police_nastenkaFP7C_actorUiRQ23std46vector<P7C_actor,Q23std20allocator<P7C_actor>>bPv |
| 0x0036fd60 | 228 | 13 | ai_police_action | {ctor} | __ct__16ai_police_actionFRC16ai_police_action |
| 0x0036fe50 | 300 | 13 | gif::vector<16ai_police_action,8> | reserve | reserve__Q23gif28vector<16ai_police_action,8>Fi |
| 0x0036ff80 | 236 | GLOBAL | ai_police_nastenka | AkceDokoncenaPlukovniku | AkceDokoncenaPlukovniku__18ai_police_nastenkaFP16ai_police_action |
| 0x00370070 | 228 | 13 | ai_police_action | operator= | __as__16ai_police_actionFRC16ai_police_action |
| 0x00370160 | 528 | GLOBAL | ai_police_nastenka | ZinvalidujAktora | ZinvalidujAktora__18ai_police_nastenkaFP7C_actor |
| 0x00370370 | 164 | GLOBAL | ai_police_nastenka | Tick | Tick__18ai_police_nastenkaFUi |
| 0x00370420 | 16 | GLOBAL | ai_police_nastenka | PlukovnikuMasVChladkuPlajeraFrafera | PlukovnikuMasVChladkuPlajeraFrafera__18ai_police_nastenkaFv |
| 0x00370430 | 348 | GLOBAL | ai_police_nastenka | PlukovnikuRadsiHoSejmemNebudemSeSrat | PlukovnikuRadsiHoSejmemNebudemSeSrat__18ai_police_nastenkaFP7C_actor |
| 0x00370590 | 364 | GLOBAL | ai_police_nastenka | PlukovnikuRadsiHoZatknemNebudemSeSrat | PlukovnikuRadsiHoZatknemNebudemSeSrat__18ai_police_nastenkaFP7C_actor |
| 0x00370700 | 160 | GLOBAL | ai_police_nastenka | MamHoZmrdaUzHoNechte | MamHoZmrdaUzHoNechte__18ai_police_nastenkaFP7C_actorP7C_actor |
| 0x003707a0 | 28 | GLOBAL | ai_police_nastenka | PlukovnikuMamZmrdaPlayera | PlukovnikuMamZmrdaPlayera__18ai_police_nastenkaFv |
| 0x003707c0 | 252 | GLOBAL | ai_police_nastenka | PanoveZmenaZatkneteHoZmrda | PanoveZmenaZatkneteHoZmrda__18ai_police_nastenkaFP7C_actor |
| 0x003708c0 | 244 | GLOBAL | ai_police_nastenka | PanoveZmenaZabijteHoZmrda | PanoveZmenaZabijteHoZmrda__18ai_police_nastenkaFP7C_actor |
| 0x003709c0 | 1376 | GLOBAL | ai_police_nastenka | PanoveKokotPremistujePrdel | PanoveKokotPremistujePrdel__18ai_police_nastenkaFP7C_actorP7C_actorb |
| 0x00370f20 | 140 | GLOBAL | ai_police_nastenka | PanoveViditeZmrdaNeboSteTupci | PanoveViditeZmrdaNeboSteTupci__18ai_police_nastenkaFP7C_actor |
| 0x00370fb0 | 120 | GLOBAL | ai_police_nastenka | PanoveVytahneteBlokyAZapisteSiZmrda | PanoveVytahneteBlokyAZapisteSiZmrda__18ai_police_nastenkaFP7C_actor |
| 0x00371030 | 292 | GLOBAL | ai_police_nastenka | PanoveMateOciNaZmrdoviNeboCumitePoBabach | PanoveMateOciNaZmrdoviNeboCumitePoBabach__18ai_police_nastenkaFP7C_actor |
| 0x00371160 | 124 | GLOBAL | ai_police_nastenka | PanoveJdeteNekdoPoZmrdovi | PanoveJdeteNekdoPoZmrdovi__18ai_police_nastenkaFP7C_actorb |
| 0x003711e0 | 156 | GLOBAL | ai_police_nastenka | PanoveJaUzHoVarovalZmetka | PanoveJaUzHoVarovalZmetka__18ai_police_nastenkaFP7C_actorP7C_actor |
| 0x00371280 | 12 | GLOBAL | ai_police_nastenka | PanoveHrajeteSiNaPoldy | PanoveHrajeteSiNaPoldy__18ai_police_nastenkaFv |
| 0x00371290 | 8 | GLOBAL | ai_police_nastenka | MakamSefe | MakamSefe__18ai_police_nastenkaFb |
| 0x003712a0 | 188 | GLOBAL | ai_police_nastenka | SemTrefenaLocklaKurva | SemTrefenaLocklaKurva__18ai_police_nastenkaFP7C_actor |
| 0x00371360 | 360 | GLOBAL | ai_police_nastenka | VseckyUbozakyStopni | VseckyUbozakyStopni__18ai_police_nastenkaFP7C_actorP7C_actor |
| 0x003714d0 | 324 | GLOBAL | ai_police_nastenka | VseckyUbozakyRozejdi | VseckyUbozakyRozejdi__18ai_police_nastenkaFP7C_actorP7C_actor |
| 0x00371620 | 108 | GLOBAL | ai_police_nastenka | HosiPrkynkujete | HosiPrkynkujete__18ai_police_nastenkaFP7C_actor |
| 0x00371690 | 8 | GLOBAL | ai_police_nastenka | TuMasZakaznika | TuMasZakaznika__18ai_police_nastenkaFP8C_entity |
| 0x003716a0 | 128 | GLOBAL | ai_police_nastenka | KonecSmeny | KonecSmeny__18ai_police_nastenkaFv |
| 0x00371720 | 348 | GLOBAL | ai_police_nastenka | ZastavMotorizovaneJednotky | ZastavMotorizovaneJednotky__18ai_police_nastenkaFv |
| 0x00371880 | 8 | GLOBAL | ai_police_nastenka | SrackyPryc | SrackyPryc__18ai_police_nastenkaFv |
| 0x00371890 | 200 | GLOBAL | ai_animitem_manager | Clear | Clear__19ai_animitem_managerFv |
| 0x00371960 | 8 | GLOBAL | ai_animitem_manager | Init | Init__19ai_animitem_managerFv |
| 0x00371970 | 132 | WEAK | std::__vector_imp<12ai_anim_item,Q23std25allocator<12ai_anim_item>,0> | erase | erase__Q23std64__vector_imp<12ai_anim_item,Q23std25allocator<12ai_anim_item>,0>FP12ai_anim_item |
| 0x00371a00 | 184 | GLOBAL | check_seg | existIndex | existIndex__9check_segFUiUiUi |
| 0x00371ac0 | 16 | GLOBAL | check_seg | NumSpecials | NumSpecials__9check_segFv |
| 0x00371ad0 | 24 | GLOBAL | check_seg | GetType | GetType__9check_segFPUiPPv |
| 0x00371af0 | 48 | GLOBAL | check_seg | GetSpecial | GetSpecial__9check_segFUiPUiPPv |
| 0x00371b20 | 652 | GLOBAL | check_seg | isInSpecial | isInSpecial__9check_segFR8S_vector |
| 0x00371db0 | 92 | GLOBAL | check_seg_clusters | GetSegment | GetSegment__18check_seg_clustersFUi |
| 0x00371e10 | 104 | GLOBAL | check_seg_clusters | NumSegments | NumSegments__18check_seg_clustersFv |
| 0x00371e80 | 1068 | GLOBAL | check_seg_clusters | AddSegment | AddSegment__18check_seg_clustersFP9check_seg |
| 0x003722b0 | 96 | 13 | check_seg_cluster | {dtor} | __dt__17check_seg_clusterFv |
| 0x00372310 | 104 | 13 | check_seg | {dtor} | __dt__9check_segFv |
| 0x00372380 | 64 | 13 | check_seg_cluster | {ctor} | __ct__17check_seg_clusterFv |
| 0x003723c0 | 68 | 13 | check_seg | {ctor} | __ct__9check_segFv |
| 0x00372410 | 228 | GLOBAL | check_seg_clusters | Init | Init__18check_seg_clustersFv |
| 0x00372500 | 528 | GLOBAL | check_seg_clusters | Clear | Clear__18check_seg_clustersFv |
| 0x00372710 | 1708 | GLOBAL | check_seg_clusters | Load | Load__18check_seg_clustersFv |
| 0x00372dc0 | 120 | GLOBAL |  | InitProgressScreen__Fv | InitProgressScreen__Fv |
| 0x00372e40 | 372 | GLOBAL |  | cbLoadScene__F21LS3D_CALLBACK_MESSAGEUiUiPv | cbLoadScene__F21LS3D_CALLBACK_MESSAGEUiUiPv |
| 0x00372fc0 | 96 | GLOBAL |  | SetWBuffer__Fb | SetWBuffer__Fb |
| 0x00373020 | 128 | GLOBAL |  | cbScene__F20SCN_CALLBACK_MESSAGEUiUi | cbScene__F20SCN_CALLBACK_MESSAGEUiUi |
| 0x003730a0 | 484 | GLOBAL |  | InternMission_Vykur__Fv | InternMission_Vykur__Fv |
| 0x00373290 | 1740 | GLOBAL |  | InternMission_Spolkni__Fv | InternMission_Spolkni__Fv |
| 0x00373960 | 2288 | GLOBAL |  | InternMissionInit_InterCity__FR11G_Inventory | InternMissionInit_InterCity__FR11G_Inventory |
| 0x00374250 | 3636 | GLOBAL |  | InternMissionInit__Fv | InternMissionInit__Fv |
| 0x00375090 | 1140 | GLOBAL |  | GameLoopSingle__Fv | GameLoopSingle__Fv |
| 0x00375510 | 624 | LOCAL |  | InitErrorMessageBox__F11LS3D_RESULTi | InitErrorMessageBox__F11LS3D_RESULTi |
| 0x00375780 | 5216 | LOCAL |  | InitSystem__FPvRb | InitSystem__FPvRb |
| 0x00376be0 | 164 | 13 | C_I3D_cache<17I3D_animation_set> | DelDir | DelDir__32C_I3D_cache<17I3D_animation_set>FPCc |
| 0x00376c90 | 164 | 13 | C_I3D_cache<9I3D_sound> | DelDir | DelDir__23C_I3D_cache<9I3D_sound>FPCc |
| 0x00376d40 | 164 | 13 | C_I3D_cache<9I3D_model> | DelDir | DelDir__23C_I3D_cache<9I3D_model>FPCc |
| 0x00376df0 | 600 | LOCAL |  | CloseSystem__Fv | CloseSystem__Fv |
| 0x00377050 | 652 | GLOBAL |  | MessageBoxGraph__FPcPcPcPcPc | MessageBoxGraph__FPcPcPcPcPc |
| 0x003772e0 | 84 | GLOBAL |  | FullAllocAndFree__Fv | FullAllocAndFree__Fv |
| 0x00377340 | 2568 | GLOBAL |  | main | main |
| 0x00377d50 | 8 | 13 | C_I3D_anim_cache | Duplicate | Duplicate__16C_I3D_anim_cacheFP17I3D_animation_setP17I3D_animation_set |
| 0x00377d60 | 20 | 13 | C_I3D_anim_cache | CreateNew | CreateNew__16C_I3D_anim_cacheCFv |
| 0x00377d80 | 180 | 13 | C_I3D_cache<17I3D_animation_set> | AddDir | AddDir__32C_I3D_cache<17I3D_animation_set>FPCc |
| 0x00377e40 | 120 | 13 | C_I3D_cache<17I3D_animation_set> | Clear | Clear__32C_I3D_cache<17I3D_animation_set>Fv |
| 0x00377ec0 | 68 | 13 | C_silver_screen | {dtor} | __dt__15C_silver_screenFv |
| 0x00377f10 | 140 | 13 | std::vector<P5C_Joy,Q23std18allocator<P5C_Joy>> | {dtor} | __dt__Q23std42vector<P5C_Joy,Q23std18allocator<P5C_Joy>>Fv |
| 0x00377fa0 | 196 | 13 | C_I3D_anim_cache | {dtor} | __dt__16C_I3D_anim_cacheFv |
| 0x00378070 | 180 | 13 | C_I3D_cache<17I3D_animation_set> | {dtor} | __dt__32C_I3D_cache<17I3D_animation_set>Fv |
| 0x00378130 | 20 | 13 | std::__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>> | {ctor} | __ct__Q23std122__vector_deleter<Q232C_I3D_cache<17I3D_animation_set>5S_itm,Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>>Fv |
| 0x00378150 | 76 | 13 | C_I3D_cache_base<17I3D_animation_set> | {dtor} | __dt__37C_I3D_cache_base<17I3D_animation_set>Fv |
| 0x003781a0 | 196 | 13 | C_I3D_sound_cache | {dtor} | __dt__17C_I3D_sound_cacheFv |
| 0x00378270 | 136 | 13 | C_I3D_cache<9I3D_sound> | Clear | Clear__23C_I3D_cache<9I3D_sound>Fv |
| 0x00378300 | 180 | 13 | C_I3D_cache<9I3D_sound> | {dtor} | __dt__23C_I3D_cache<9I3D_sound>Fv |
| 0x003783c0 | 20 | 13 | std::__vector_deleter<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>> | {ctor} | __ct__Q23std104__vector_deleter<Q223C_I3D_cache<9I3D_sound>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>>Fv |
| 0x003783e0 | 76 | 13 | C_I3D_cache_base<9I3D_sound> | {dtor} | __dt__28C_I3D_cache_base<9I3D_sound>Fv |
| 0x00378430 | 196 | 13 | C_I3D_model_cache | {dtor} | __dt__17C_I3D_model_cacheFv |
| 0x00378500 | 136 | 13 | C_I3D_cache<9I3D_model> | Clear | Clear__23C_I3D_cache<9I3D_model>Fv |
| 0x00378590 | 180 | 13 | C_I3D_cache<9I3D_model> | {dtor} | __dt__23C_I3D_cache<9I3D_model>Fv |
| 0x00378650 | 20 | 13 | std::__vector_deleter<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>> | {ctor} | __ct__Q23std104__vector_deleter<Q223C_I3D_cache<9I3D_model>5S_itm,Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>>Fv |
| 0x00378670 | 76 | 13 | C_I3D_cache_base<9I3D_model> | {dtor} | __dt__28C_I3D_cache_base<9I3D_model>Fv |
| 0x003786c0 | 108 | 13 | C_I3D_sound_cache | Duplicate | Duplicate__17C_I3D_sound_cacheFP9I3D_soundP9I3D_sound |
| 0x00378730 | 24 | 13 | C_I3D_sound_cache | CreateNew | CreateNew__17C_I3D_sound_cacheCFv |
| 0x00378750 | 180 | 13 | C_I3D_cache<9I3D_sound> | AddDir | AddDir__23C_I3D_cache<9I3D_sound>FPCc |
| 0x00378810 | 88 | 13 | C_I3D_model_cache | Duplicate | Duplicate__17C_I3D_model_cacheFP9I3D_modelP9I3D_model |
| 0x00378870 | 16 | 13 | C_I3D_model_cache | cbNullData | cbNullData__17C_I3D_model_cacheFP9I3D_frameUi |
| 0x00378880 | 24 | 13 | C_I3D_model_cache | CreateNew | CreateNew__17C_I3D_model_cacheCFv |
| 0x003788a0 | 180 | 13 | C_I3D_cache<9I3D_model> | AddDir | AddDir__23C_I3D_cache<9I3D_model>FPCc |
| 0x00378960 | 8 | WEAK | C_I3D_cache<17I3D_animation_set> | SetCacheSize | SetCacheSize__32C_I3D_cache<17I3D_animation_set>Fi |
| 0x00378970 | 96 | WEAK | C_I3D_cache<17I3D_animation_set> | ClearDirs | ClearDirs__32C_I3D_cache<17I3D_animation_set>Fv |
| 0x003789d0 | 8 | WEAK | C_I3D_cache<17I3D_animation_set> | NumDirs | NumDirs__32C_I3D_cache<17I3D_animation_set>CFv |
| 0x003789e0 | 8 | WEAK | C_I3D_cache<9I3D_sound> | SetCacheSize | SetCacheSize__23C_I3D_cache<9I3D_sound>Fi |
| 0x003789f0 | 96 | WEAK | C_I3D_cache<9I3D_sound> | ClearDirs | ClearDirs__23C_I3D_cache<9I3D_sound>Fv |
| 0x00378a50 | 8 | WEAK | C_I3D_cache<9I3D_sound> | NumDirs | NumDirs__23C_I3D_cache<9I3D_sound>CFv |
| 0x00378a60 | 8 | WEAK | C_I3D_cache<9I3D_model> | SetCacheSize | SetCacheSize__23C_I3D_cache<9I3D_model>Fi |
| 0x00378a70 | 792 | WEAK | C_I3D_cache<9I3D_model> | Create | Create__23C_I3D_cache<9I3D_model>FPCcPP9I3D_modelPFPCcPv_vPv |
| 0x00378d90 | 96 | WEAK | C_I3D_cache<9I3D_model> | ClearDirs | ClearDirs__23C_I3D_cache<9I3D_model>Fv |
| 0x00378df0 | 8 | WEAK | C_I3D_cache<9I3D_model> | NumDirs | NumDirs__23C_I3D_cache<9I3D_model>CFv |
| 0x00378e00 | 20 | WEAK | C_I3D_cache<9I3D_model> | GetDir | GetDir__23C_I3D_cache<9I3D_model>CFi |
| 0x00378e20 | 160 | WEAK | C_I3D_cache<9I3D_model> | ClearUpper | ClearUpper__23C_I3D_cache<9I3D_model>Fv |
| 0x00378ec0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>,Ui,1> | first | first__Q310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_model>5S_itm>,Ui,1>Fv |
| 0x00378ed0 | 20 | WEAK | C_I3D_cache<9I3D_sound> | GetDir | GetDir__23C_I3D_cache<9I3D_sound>CFi |
| 0x00378ef0 | 160 | WEAK | C_I3D_cache<9I3D_sound> | ClearUpper | ClearUpper__23C_I3D_cache<9I3D_sound>Fv |
| 0x00378f90 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>,Ui,1> | first | first__Q310Metrowerks7details78compressed_pair_imp<Q23std44allocator<Q223C_I3D_cache<9I3D_sound>5S_itm>,Ui,1>Fv |
| 0x00378fa0 | 784 | WEAK | C_I3D_cache<9I3D_sound> | Create | Create__23C_I3D_cache<9I3D_sound>FPCcPP9I3D_soundPFPCcPv_vPv |
| 0x003792b0 | 1180 | WEAK | C_I3D_cache<9I3D_sound> | Open | Open__23C_I3D_cache<9I3D_sound>FP9I3D_soundPCcUiPFPCcPv_vPvPv |
| 0x00379750 | 20 | WEAK | C_I3D_cache<17I3D_animation_set> | GetDir | GetDir__32C_I3D_cache<17I3D_animation_set>CFi |
| 0x00379770 | 152 | WEAK | C_I3D_cache<17I3D_animation_set> | ClearUpper | ClearUpper__32C_I3D_cache<17I3D_animation_set>Fv |
| 0x00379810 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>,Ui,1> | first | first__Q310Metrowerks7details87compressed_pair_imp<Q23std53allocator<Q232C_I3D_cache<17I3D_animation_set>5S_itm>,Ui,1>Fv |
| 0x00379820 | 1148 | WEAK | C_I3D_cache<17I3D_animation_set> | Open | Open__32C_I3D_cache<17I3D_animation_set>FP17I3D_animation_setPCcUiPFPCcPv_vPvPv |
| 0x00379ca0 | 664 | GLOBAL |  | MC_Process__Fv | MC_Process__Fv |
| 0x00379f40 | 116 | GLOBAL |  | MC_CheckSaveHeads__FcPviiPiPcPcii | MC_CheckSaveHeads__FcPviiPiPcPcii |
| 0x00379fc0 | 472 | GLOBAL |  | MC_GetMemoryCardsInfo__FPP14MEMORYCARDINFOPi | MC_GetMemoryCardsInfo__FPP14MEMORYCARDINFOPi |
| 0x0037a1a0 | 624 | GLOBAL |  | MC_SaveData__FPcPviiPcPci | MC_SaveData__FPcPviiPcPci |
| 0x0037a410 | 100 | GLOBAL |  | MC_DeleteData__FPcPc | MC_DeleteData__FPcPc |
| 0x0037a480 | 132 | GLOBAL |  | MC_LoadData__FPcPviPc | MC_LoadData__FPcPviPc |
| 0x0037a510 | 16 | GLOBAL |  | MC_Format__Fi | MC_Format__Fi |
| 0x0037a520 | 492 | GLOBAL | C_mission | {ctor} | __ct__9C_missionFv |
| 0x0037a710 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q29C_mission20S_actor_sound_buffer>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q29C_mission20S_actor_sound_buffer>,Ui,1>FUi |
| 0x0037a720 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q29C_mission20S_frame_actor_buffer>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q29C_mission20S_frame_actor_buffer>,Ui,1>FUi |
| 0x0037a730 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std40allocator<Q29C_mission15S_t_transparent>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details74compressed_pair_imp<Q23std40allocator<Q29C_mission15S_t_transparent>,Ui,1>FUi |
| 0x0037a740 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std37allocator<Q29C_mission12S_anim_model>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q29C_mission12S_anim_model>,Ui,1>FUi |
| 0x0037a750 | 432 | GLOBAL | C_mission | {dtor} | __dt__9C_missionFv |
| 0x0037a900 | 1476 | GLOBAL | C_mission | Init | Init__9C_missionFP10I3D_driver |
| 0x0037aed0 | 20 | 13 | std::__vector_deleter<Q26C_game15S_save_game_fql,Q23std37allocator<Q26C_game15S_save_game_fql>> | {ctor} | __ct__Q23std90__vector_deleter<Q26C_game15S_save_game_fql,Q23std37allocator<Q26C_game15S_save_game_fql>>Fv |
| 0x0037aef0 | 20 | 13 | std::__vector_deleter<Q26C_game13S_scream_area,Q23std35allocator<Q26C_game13S_scream_area>> | {ctor} | __ct__Q23std86__vector_deleter<Q26C_game13S_scream_area,Q23std35allocator<Q26C_game13S_scream_area>>Fv |
| 0x0037af10 | 20 | 13 | std::__vector_deleter<Q26C_game12S_find_actor,Q23std34allocator<Q26C_game12S_find_actor>> | {ctor} | __ct__Q23std84__vector_deleter<Q26C_game12S_find_actor,Q23std34allocator<Q26C_game12S_find_actor>>Fv |
| 0x0037af30 | 20 | 13 | std::__vector_deleter<Q26C_game12S_enemy_talk,Q23std34allocator<Q26C_game12S_enemy_talk>> | {ctor} | __ct__Q23std84__vector_deleter<Q26C_game12S_enemy_talk,Q23std34allocator<Q26C_game12S_enemy_talk>>Fv |
| 0x0037af50 | 20 | 13 | std::__vector_deleter<Q26C_game13S_i_show_fade,Q23std35allocator<Q26C_game13S_i_show_fade>> | {ctor} | __ct__Q23std86__vector_deleter<Q26C_game13S_i_show_fade,Q23std35allocator<Q26C_game13S_i_show_fade>>Fv |
| 0x0037af70 | 20 | 13 | std::__vector_deleter<Q26C_game8S_i_show,Q23std29allocator<Q26C_game8S_i_show>> | {ctor} | __ct__Q23std74__vector_deleter<Q26C_game8S_i_show,Q23std29allocator<Q26C_game8S_i_show>>Fv |
| 0x0037af90 | 20 | 13 | std::__vector_deleter<Q26C_game12S_driver_man,Q23std34allocator<Q26C_game12S_driver_man>> | {ctor} | __ct__Q23std84__vector_deleter<Q26C_game12S_driver_man,Q23std34allocator<Q26C_game12S_driver_man>>Fv |
| 0x0037afb0 | 20 | 13 | std::__vector_deleter<Q26C_game16S_npc_kill_event,Q23std38allocator<Q26C_game16S_npc_kill_event>> | {ctor} | __ct__Q23std92__vector_deleter<Q26C_game16S_npc_kill_event,Q23std38allocator<Q26C_game16S_npc_kill_event>>Fv |
| 0x0037afd0 | 104 | 13 | std::__vector_imp<PQ214C_using_object14S_using_object,Q23std46allocator<PQ214C_using_object14S_using_object>,1> | {dtor} | __dt__Q23std106__vector_imp<PQ214C_using_object14S_using_object,Q23std46allocator<PQ214C_using_object14S_using_object>,1>Fv |
| 0x0037b040 | 8 | 13 | I3D_bbox | {ctor} | __ct__8I3D_bboxFv |
| 0x0037b050 | 20 | 13 | std::__vector_deleter<Q26C_game12S_sound_play,Q23std34allocator<Q26C_game12S_sound_play>> | {ctor} | __ct__Q23std84__vector_deleter<Q26C_game12S_sound_play,Q23std34allocator<Q26C_game12S_sound_play>>Fv |
| 0x0037b070 | 20 | 13 | std::__vector_deleter<Q26C_game13S_shadow_item,Q23std35allocator<Q26C_game13S_shadow_item>> | {ctor} | __ct__Q23std86__vector_deleter<Q26C_game13S_shadow_item,Q23std35allocator<Q26C_game13S_shadow_item>>Fv |
| 0x0037b090 | 20 | 13 | std::__vector_deleter<Q26C_game13S_shadow_temp,Q23std35allocator<Q26C_game13S_shadow_temp>> | {ctor} | __ct__Q23std86__vector_deleter<Q26C_game13S_shadow_temp,Q23std35allocator<Q26C_game13S_shadow_temp>>Fv |
| 0x0037b0b0 | 104 | 13 | gif::vector<7S_FLASH,8> | {dtor} | __dt__Q23gif18vector<7S_FLASH,8>Fv |
| 0x0037b120 | 104 | 13 | gif::vector<8S_DEBRIS,8> | {dtor} | __dt__Q23gif19vector<8S_DEBRIS,8>Fv |
| 0x0037b190 | 104 | 13 | gif::vector<6S_DICE,8> | {dtor} | __dt__Q23gif17vector<6S_DICE,8>Fv |
| 0x0037b200 | 104 | 13 | gif::vector<P9I3D_frame,8> | {dtor} | __dt__Q23gif21vector<P9I3D_frame,8>Fv |
| 0x0037b270 | 20 | 13 | std::__vector_pod<Us,Q23std13allocator<Us>> | {ctor} | __ct__Q23std38__vector_pod<Us,Q23std13allocator<Us>>Fv |
| 0x0037b290 | 20 | 13 | std::__vector_deleter<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>> | {ctor} | __ct__Q23std82__vector_deleter<Q26C_game11S_Telephone,Q23std33allocator<Q26C_game11S_Telephone>>Fv |
| 0x0037b2b0 | 20 | 13 | std::__vector_deleter<Q26C_game11S_RoadBlock,Q23std33allocator<Q26C_game11S_RoadBlock>> | {ctor} | __ct__Q23std82__vector_deleter<Q26C_game11S_RoadBlock,Q23std33allocator<Q26C_game11S_RoadBlock>>Fv |
| 0x0037b2d0 | 20 | 13 | std::__vector_deleter<11S_semaphore,Q23std24allocator<11S_semaphore>> | {ctor} | __ct__Q23std64__vector_deleter<11S_semaphore,Q23std24allocator<11S_semaphore>>Fv |
| 0x0037b2f0 | 20 | 13 | std::__vector_deleter<Q26C_game17S_invisible_actor,Q23std39allocator<Q26C_game17S_invisible_actor>> | {ctor} | __ct__Q23std94__vector_deleter<Q26C_game17S_invisible_actor,Q23std39allocator<Q26C_game17S_invisible_actor>>Fv |
| 0x0037b310 | 2448 | GLOBAL | C_mission | UnInit | UnInit__9C_missionFv |
| 0x0037bca0 | 180 | GLOBAL | C_mission | AddAnimModel | AddAnimModel__9C_missionFP9I3D_modelb |
| 0x0037bd60 | 92 | GLOBAL | C_mission | DelAnimModel | DelAnimModel__9C_missionFP9I3D_model |
| 0x0037bdc0 | 148 | GLOBAL | C_mission | ClearGameAnimModels | ClearGameAnimModels__9C_missionFv |
| 0x0037be60 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std37allocator<Q29C_mission12S_anim_model>,Ui,1> | first | first__Q310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q29C_mission12S_anim_model>,Ui,1>Fv |
| 0x0037be70 | 1168 | GLOBAL | C_mission | CreateActor | CreateActor__9C_missionFi |
| 0x0037c300 | 276 | GLOBAL | C_mission | AddActor | AddActor__9C_missionFP7C_actorb |
| 0x0037c420 | 248 | GLOBAL | C_mission | UnloadActor | UnloadActor__9C_missionFP7C_actor |
| 0x0037c520 | 360 | GLOBAL | C_mission | DelActor | DelActor__9C_missionFP7C_actor |
| 0x0037c690 | 248 | GLOBAL | C_mission | DelActors | DelActors__9C_missionFv |
| 0x0037c790 | 340 | GLOBAL | C_mission | DelNotMeActors | DelNotMeActors__9C_missionFv |
| 0x0037c8f0 | 192 | GLOBAL | C_mission | FindActorByName | FindActorByName__9C_missionFPCc |
| 0x0037c9b0 | 100 | GLOBAL | C_mission | GlobalProgramGameInit | GlobalProgramGameInit__9C_missionFv |
| 0x0037ca20 | 100 | GLOBAL | C_mission | GlobalProgramGameDone | GlobalProgramGameDone__9C_missionFv |
| 0x0037ca90 | 164 | GLOBAL | C_mission | GlobalProgramRun | GlobalProgramRun__9C_missionFPc |
| 0x0037cb40 | 1176 | GLOBAL | C_mission | Close | Close__9C_missionFv |
| 0x0037cfe0 | 676 | GLOBAL | C_mission | Tick | Tick__9C_missionFi |
| 0x0037d290 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std40allocator<Q29C_mission15S_t_transparent>,Ui,1> | first | first__Q310Metrowerks7details74compressed_pair_imp<Q23std40allocator<Q29C_mission15S_t_transparent>,Ui,1>Fv |
| 0x0037d2a0 | 48 | LOCAL |  | __SceneErrFnc__F21LS3D_CALLBACK_MESSAGEUiUiPv | __SceneErrFnc__F21LS3D_CALLBACK_MESSAGEUiUiPv |
| 0x0037d2d0 | 244 | GLOBAL |  | ShowProgress__FP7C_chunk | ShowProgress__FP7C_chunk |
| 0x0037d3d0 | 3516 | GLOBAL | C_mission | Open | Open__9C_missionFPCcbUib |
| 0x0037e190 | 396 | 13 | C_mission::S_hlp$4890Mission_cpp | cbSectorBitFill | cbSectorBitFill__Q29C_mission21S_hlp$4890Mission_cppFP9I3D_frame |
| 0x0037e320 | 352 | GLOBAL |  | ReadVector3__FR7C_chunk | ReadVector3__FR7C_chunk |
| 0x0037e480 | 384 | GLOBAL |  | ReadQuatOld__FR7C_chunk | ReadQuatOld__FR7C_chunk |
| 0x0037e600 | 460 | GLOBAL |  | ReadBBoxOld__FR7C_chunkR8I3D_bbox | ReadBBoxOld__FR7C_chunkR8I3D_bbox |
| 0x0037e7d0 | 104 | GLOBAL |  | DelPicaModelDel__Fv | DelPicaModelDel__Fv |
| 0x0037e840 | 30632 | GLOBAL | C_mission | OpenBin | OpenBin__9C_missionFPCcUi |
| 0x00385ff0 | 8 | 13 | std::__vector_deleter<13S_tshift_elem,Q23std26allocator<13S_tshift_elem>> | data | data__Q23std68__vector_deleter<13S_tshift_elem,Q23std26allocator<13S_tshift_elem>>Fv |
| 0x00386000 | 264 | 13 | C_str | {ctor} | __ct__5C_strFPCce |
| 0x00386110 | 188 | 13 | C_str | operator= | __as__5C_strFPCc |
| 0x003861d0 | 216 | 13 | C_chunk | Read | Read__7C_chunkFPvi |
| 0x003862b0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std33allocator<20S_sound_light_sector>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details67compressed_pair_imp<Q23std33allocator<20S_sound_light_sector>,Ui,1>FUi |
| 0x003862c0 | 164 | GLOBAL | C_mission | AddTimeTransparentObject | AddTimeTransparentObject__9C_missionFP10I3D_objects |
| 0x00386370 | 152 | GLOBAL | C_mission | FinishTimeTransparentObjects | FinishTimeTransparentObjects__9C_missionFv |
| 0x00386410 | 260 | GLOBAL | C_mission | UpdateSettings | UpdateSettings__9C_missionFv |
| 0x00386520 | 124 | GLOBAL | C_mission | RestoreSettings | RestoreSettings__9C_missionFv |
| 0x003865a0 | 132 | LOCAL |  | ClearCarSeats__FP5C_car | ClearCarSeats__FP5C_car |
| 0x00386630 | 484 | GLOBAL | C_mission | FrmActBuff_StoreActor | FrmActBuff_StoreActor__9C_missionFP7C_actorb |
| 0x00386820 | 248 | GLOBAL | C_mission | FrmActBuff_StoreModel | FrmActBuff_StoreModel__9C_missionFP9I3D_model |
| 0x00386920 | 384 | GLOBAL | C_mission | FrmActBuff_GetActor | FrmActBuff_GetActor__9C_missionFPci |
| 0x00386aa0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q29C_mission20S_frame_actor_buffer>,Ui,1> | first | first__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q29C_mission20S_frame_actor_buffer>,Ui,1>Fv |
| 0x00386ab0 | 260 | GLOBAL | C_mission | FrmActBuff_GetModel | FrmActBuff_GetModel__9C_missionFPc |
| 0x00386bc0 | 236 | GLOBAL | C_mission | FrmActBuff_StoreActiveActor | FrmActBuff_StoreActiveActor__9C_missionFP7C_actor |
| 0x00386cb0 | 172 | GLOBAL | C_mission | FrmActBuff_RemoveActiveActor | FrmActBuff_RemoveActiveActor__9C_missionFP7C_actor |
| 0x00386d60 | 268 | GLOBAL | C_mission | FrmActBuff_IsActiveActor | FrmActBuff_IsActiveActor__9C_missionFP7C_actor |
| 0x00386e70 | 152 | GLOBAL | C_mission | FrmActBuff_StoreActorSound | FrmActBuff_StoreActorSound__9C_missionFP7C_actoriP9I3D_sound |
| 0x00386f10 | 172 | GLOBAL | C_mission | FrmActBuff_GetActorSound | FrmActBuff_GetActorSound__9C_missionFP7C_actori |
| 0x00386fc0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q29C_mission20S_actor_sound_buffer>,Ui,1> | first | first__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q29C_mission20S_actor_sound_buffer>,Ui,1>Fv |
| 0x00386fd0 | 392 | GLOBAL | C_mission | FrmActBuff_ClearAll | FrmActBuff_ClearAll__9C_missionFv |
| 0x00387160 | 220 | GLOBAL |  | ReleaseUniqueTextures__Fv | ReleaseUniqueTextures__Fv |
| 0x00387240 | 104 | GLOBAL | C_mission | ClearFramesRefBuff | ClearFramesRefBuff__9C_missionFv |
| 0x003872b0 | 6120 | GLOBAL |  | LoadActorFromIntercity__FR7C_chunk | LoadActorFromIntercity__FR7C_chunk |
| 0x00388aa0 | 1672 | GLOBAL | C_mission | InterCity_Prebuffering | InterCity_Prebuffering__9C_missionFPc |
| 0x00389130 | 3232 | GLOBAL | C_mission | InterCity_Init | InterCity_Init__9C_missionFPc |
| 0x00389dd0 | 120 | GLOBAL | C_mission | InterCity_PicaDelActors | InterCity_PicaDelActors__9C_missionFv |
| 0x00389e50 | 544 | GLOBAL | C_mission | InterCity_Done | InterCity_Done__9C_missionFv |
| 0x0038a070 | 936 | GLOBAL | C_mission | InterCity_GameInit | InterCity_GameInit__9C_missionFv |
| 0x0038a420 | 148 | GLOBAL | C_mission | InterCity_GameDone | InterCity_GameDone__9C_missionFv |
| 0x0038a4c0 | 864 | GLOBAL | C_mission | InterCity_GameDoneInGameDone | InterCity_GameDoneInGameDone__9C_missionFv |
| 0x0038a820 | 180 | GLOBAL |  | SmrdiZaPrdeli__FP7C_actorP7C_actor | SmrdiZaPrdeli__FP7C_actorP7C_actor |
| 0x0038a8e0 | 3344 | GLOBAL | C_mission | InterCity_ActorsToMission | InterCity_ActorsToMission__9C_missionFb |
| 0x0038b5f0 | 100 | GLOBAL | C_mission | InterCity_ActorsFromMission | InterCity_ActorsFromMission__9C_missionFv |
| 0x0038b660 | 1768 | GLOBAL | C_mission | InterCity_ChangeMission | InterCity_ChangeMission__9C_missionFv |
| 0x0038bd50 | 8 | GLOBAL | C_mission | InterCity_SetMissionName | InterCity_SetMissionName__9C_missionFPc |
| 0x0038bd60 | 116 | GLOBAL | C_mission | InterCity_RepairDirName | InterCity_RepairDirName__9C_missionFPc |
| 0x0038bde0 | 136 | GLOBAL | C_mission | InterCity_AddGlobalActor | InterCity_AddGlobalActor__9C_missionFP7C_actor |
| 0x0038be70 | 96 | GLOBAL | C_mission | InterCity_DelGlobalActor | InterCity_DelGlobalActor__9C_missionFP7C_actor |
| 0x0038bed0 | 76 | GLOBAL | C_mission | IsInterCityGlobalActor | IsInterCityGlobalActor__9C_missionFP7C_actor |
| 0x0038bf20 | 160 | GLOBAL | C_mission | LoadFQLPath | LoadFQLPath__9C_missionFv |
| 0x0038bfc0 | 44 | GLOBAL | C_mission | UnloadFQLPath | UnloadFQLPath__9C_missionFv |
| 0x0038bff0 | 208 | GLOBAL | C_mission | GetFQLPath | GetFQLPath__9C_missionFPc |
| 0x0038c0c0 | 160 | GLOBAL | C_mission | LoadAnimationPath | LoadAnimationPath__9C_missionFv |
| 0x0038c160 | 44 | GLOBAL | C_mission | UnloadAnimationPath | UnloadAnimationPath__9C_missionFv |
| 0x0038c190 | 200 | GLOBAL | C_mission | GetAnimationPath | GetAnimationPath__9C_missionFPc |
| 0x0038c260 | 100 | GLOBAL | C_mission | InterCity_SaveLoadInit | InterCity_SaveLoadInit__9C_missionFv |
| 0x0038c2d0 | 216 | GLOBAL | C_mission | InterCity_SaveLoadDone | InterCity_SaveLoadDone__9C_missionFv |
| 0x0038c3b0 | 412 | GLOBAL | C_mission | InterCity_RepairActorActState | InterCity_RepairActorActState__9C_missionFv |
| 0x0038c550 | 136 | GLOBAL | C_mission | InterCity_AddTempAct | InterCity_AddTempAct__9C_missionFP7C_actor |
| 0x0038c5e0 | 88 | GLOBAL | C_mission | InterCity_DelTempAct | InterCity_DelTempAct__9C_missionFP7C_actor |
| 0x0038c640 | 588 | GLOBAL | C_mission | InterCity_TickTempAct | InterCity_TickTempAct__9C_missionFv |
| 0x0038c890 | 284 | GLOBAL | C_mission | Garage_AddCar | Garage_AddCar__9C_missionFP5C_car |
| 0x0038c9b0 | 1064 | GLOBAL | C_mission | Garage_DelCar | Garage_DelCar__9C_missionFP5C_car |
| 0x0038cde0 | 1948 | GLOBAL | C_mission | Freeride_ChangeCar | Freeride_ChangeCar__9C_missionFP5C_carUi |
| 0x0038d580 | 396 | GLOBAL |  | VyjebajVsechnySklaAleNeporezSiHo__Fv | VyjebajVsechnySklaAleNeporezSiHo__Fv |
| 0x0038d710 | 120 | GLOBAL | C_mission | OpenFQLFile | OpenFQLFile__9C_missionFPc |
| 0x0038d790 | 60 | GLOBAL | C_mission | CloseFQLFile | CloseFQLFile__9C_missionFv |
| 0x0038d7d0 | 80 | WEAK | std::__vector_deleter<Q29C_mission20S_actor_sound_buffer,Q23std45allocator<Q29C_mission20S_actor_sound_buffer>> | clear | clear__Q23std106__vector_deleter<Q29C_mission20S_actor_sound_buffer,Q23std45allocator<Q29C_mission20S_actor_sound_buffer>>Fv |
| 0x0038d820 | 72 | WEAK | std::__vector_deleter<Q29C_mission20S_frame_actor_buffer,Q23std45allocator<Q29C_mission20S_frame_actor_buffer>> | clear | clear__Q23std106__vector_deleter<Q29C_mission20S_frame_actor_buffer,Q23std45allocator<Q29C_mission20S_frame_actor_buffer>>Fv |
| 0x0038d870 | 72 | WEAK | std::__vector_deleter<Q29C_mission15S_t_transparent,Q23std40allocator<Q29C_mission15S_t_transparent>> | clear | clear__Q23std96__vector_deleter<Q29C_mission15S_t_transparent,Q23std40allocator<Q29C_mission15S_t_transparent>>Fv |
| 0x0038d8c0 | 72 | WEAK | std::__vector_deleter<Q29C_mission12S_anim_model,Q23std37allocator<Q29C_mission12S_anim_model>> | clear | clear__Q23std90__vector_deleter<Q29C_mission12S_anim_model,Q23std37allocator<Q29C_mission12S_anim_model>>Fv |
| 0x0038d910 | 116 | WEAK | std::__vector_imp<Q29C_mission12S_anim_model,Q23std37allocator<Q29C_mission12S_anim_model>,0> | erase | erase__Q23std88__vector_imp<Q29C_mission12S_anim_model,Q23std37allocator<Q29C_mission12S_anim_model>,0>FPQ29C_mission12S_anim_model |
| 0x0038d990 | 1052 | WEAK | std::__vector_imp<Q29C_mission12S_anim_model,Q23std37allocator<Q29C_mission12S_anim_model>,0> | insert | insert__Q23std88__vector_imp<Q29C_mission12S_anim_model,Q23std37allocator<Q29C_mission12S_anim_model>,0>FPQ29C_mission12S_anim_modelUiRCQ29C_mission12S_anim_model |
| 0x0038ddb0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std37allocator<Q29C_mission12S_anim_model>,Ui,1> | swap | swap__Q310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q29C_mission12S_anim_model>,Ui,1>FRQ310Metrowerks7details71compressed_pair_imp<Q23std37allocator<Q29C_mission12S_anim_model>,Ui,1> |
| 0x0038ddd0 | 1068 | WEAK | std::__vector_imp<Q29C_mission15S_t_transparent,Q23std40allocator<Q29C_mission15S_t_transparent>,0> | insert | insert__Q23std94__vector_imp<Q29C_mission15S_t_transparent,Q23std40allocator<Q29C_mission15S_t_transparent>,0>FPQ29C_mission15S_t_transparentUiRCQ29C_mission15S_t_transparent |
| 0x0038e200 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std40allocator<Q29C_mission15S_t_transparent>,Ui,1> | swap | swap__Q310Metrowerks7details74compressed_pair_imp<Q23std40allocator<Q29C_mission15S_t_transparent>,Ui,1>FRQ310Metrowerks7details74compressed_pair_imp<Q23std40allocator<Q29C_mission15S_t_transparent>,Ui,1> |
| 0x0038e220 | 1180 | WEAK | std::__vector_imp<Q29C_mission20S_frame_actor_buffer,Q23std45allocator<Q29C_mission20S_frame_actor_buffer>,0> | insert | insert__Q23std104__vector_imp<Q29C_mission20S_frame_actor_buffer,Q23std45allocator<Q29C_mission20S_frame_actor_buffer>,0>FPQ29C_mission20S_frame_actor_bufferUiRCQ29C_mission20S_frame_actor_buffer |
| 0x0038e6c0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q29C_mission20S_frame_actor_buffer>,Ui,1> | swap | swap__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q29C_mission20S_frame_actor_buffer>,Ui,1>FRQ310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q29C_mission20S_frame_actor_buffer>,Ui,1> |
| 0x0038e6e0 | 1172 | WEAK | std::__vector_imp<Q29C_mission20S_actor_sound_buffer,Q23std45allocator<Q29C_mission20S_actor_sound_buffer>,0> | insert | insert__Q23std104__vector_imp<Q29C_mission20S_actor_sound_buffer,Q23std45allocator<Q29C_mission20S_actor_sound_buffer>,0>FPQ29C_mission20S_actor_sound_bufferUiRCQ29C_mission20S_actor_sound_buffer |
| 0x0038eb80 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std45allocator<Q29C_mission20S_actor_sound_buffer>,Ui,1> | swap | swap__Q310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q29C_mission20S_actor_sound_buffer>,Ui,1>FRQ310Metrowerks7details79compressed_pair_imp<Q23std45allocator<Q29C_mission20S_actor_sound_buffer>,Ui,1> |
| 0x0038eba0 | 1388 | WEAK | std::__vector_imp<13S_tshift_elem,Q23std26allocator<13S_tshift_elem>,0> | insert | insert__Q23std66__vector_imp<13S_tshift_elem,Q23std26allocator<13S_tshift_elem>,0>FP13S_tshift_elemUiRC13S_tshift_elem |
| 0x0038f110 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_tshift_elem>,Ui,1> | swap | swap__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_tshift_elem>,Ui,1>FRQ310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_tshift_elem>,Ui,1> |
| 0x0038f130 | 64 | 13 | S_tshift_elem | operator= | __as__13S_tshift_elemFRC13S_tshift_elem |
| 0x0038f170 | 1052 | WEAK | std::__vector_imp<20S_sound_light_sector,Q23std33allocator<20S_sound_light_sector>,0> | insert | insert__Q23std80__vector_imp<20S_sound_light_sector,Q23std33allocator<20S_sound_light_sector>,0>FP20S_sound_light_sectorUiRC20S_sound_light_sector |
| 0x0038f590 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std33allocator<20S_sound_light_sector>,Ui,1> | swap | swap__Q310Metrowerks7details67compressed_pair_imp<Q23std33allocator<20S_sound_light_sector>,Ui,1>FRQ310Metrowerks7details67compressed_pair_imp<Q23std33allocator<20S_sound_light_sector>,Ui,1> |
| 0x0038f5b0 | 104 | WEAK | std::__vector_deleter<Q29C_mission12S_anim_model,Q23std37allocator<Q29C_mission12S_anim_model>> | {dtor} | __dt__Q23std90__vector_deleter<Q29C_mission12S_anim_model,Q23std37allocator<Q29C_mission12S_anim_model>>Fv |
| 0x0038f620 | 104 | WEAK | std::__vector_deleter<Q29C_mission15S_t_transparent,Q23std40allocator<Q29C_mission15S_t_transparent>> | {dtor} | __dt__Q23std96__vector_deleter<Q29C_mission15S_t_transparent,Q23std40allocator<Q29C_mission15S_t_transparent>>Fv |
| 0x0038f690 | 104 | WEAK | std::__vector_deleter<Q29C_mission20S_frame_actor_buffer,Q23std45allocator<Q29C_mission20S_frame_actor_buffer>> | {dtor} | __dt__Q23std106__vector_deleter<Q29C_mission20S_frame_actor_buffer,Q23std45allocator<Q29C_mission20S_frame_actor_buffer>>Fv |
| 0x0038f700 | 104 | WEAK | std::__vector_deleter<Q29C_mission20S_actor_sound_buffer,Q23std45allocator<Q29C_mission20S_actor_sound_buffer>> | {dtor} | __dt__Q23std106__vector_deleter<Q29C_mission20S_actor_sound_buffer,Q23std45allocator<Q29C_mission20S_actor_sound_buffer>>Fv |
| 0x0038f770 | 80 | WEAK | std::__vector_deleter<13S_tshift_elem,Q23std26allocator<13S_tshift_elem>> | clear | clear__Q23std68__vector_deleter<13S_tshift_elem,Q23std26allocator<13S_tshift_elem>>Fv |
| 0x0038f7c0 | 104 | WEAK | std::__vector_deleter<20S_sound_light_sector,Q23std33allocator<20S_sound_light_sector>> | {dtor} | __dt__Q23std82__vector_deleter<20S_sound_light_sector,Q23std33allocator<20S_sound_light_sector>>Fv |
| 0x0038f830 | 104 | WEAK | std::__vector_deleter<13S_tshift_elem,Q23std26allocator<13S_tshift_elem>> | {dtor} | __dt__Q23std68__vector_deleter<13S_tshift_elem,Q23std26allocator<13S_tshift_elem>>Fv |
| 0x0038f8a0 | 72 | WEAK | std::__vector_deleter<20S_sound_light_sector,Q23std33allocator<20S_sound_light_sector>> | clear | clear__Q23std82__vector_deleter<20S_sound_light_sector,Q23std33allocator<20S_sound_light_sector>>Fv |
| 0x0038f8f0 | 104 | 13 | gif::vector<P8ITexture,8> | {dtor} | __dt__Q23gif20vector<P8ITexture,8>Fv |
| 0x0038f960 | 140 | 13 | std::vector<P9I3D_model,Q23std22allocator<P9I3D_model>> | {dtor} | __dt__Q23std50vector<P9I3D_model,Q23std22allocator<P9I3D_model>>Fv |
| 0x0038f9f0 | 68 | GLOBAL |  | OpenMenuScene__Fv | OpenMenuScene__Fv |
| 0x0038fa40 | 128 | GLOBAL |  | PlayMovie__Fb | PlayMovie__Fb |
| 0x0038fac0 | 1172 | GLOBAL |  | GameLoopSingle1__Fv | GameLoopSingle1__Fv |
| 0x0038ff60 | 780 | GLOBAL |  | RunRacing__FRUi | RunRacing__FRUi |
| 0x00390270 | 480 | GLOBAL |  | ExecuteRacingMenu__Fv | ExecuteRacingMenu__Fv |
| 0x00390450 | 268 | GLOBAL |  | RacingMisse06__FUiUiUiUi | RacingMisse06__FUiUiUiUi |
| 0x00390560 | 140 | GLOBAL |  | RacingMisse06ExecuteLoop__Fv | RacingMisse06ExecuteLoop__Fv |
| 0x003905f0 | 68 | GLOBAL | C_web_node | {dtor} | __dt__10C_web_nodeFv |
| 0x00390640 | 264 | GLOBAL | C_web_node | Resize | Resize__10C_web_nodeFUsP10C_web_path |
| 0x00390750 | 68 | 13 | C_web_connection | {dtor} | __dt__16C_web_connectionFv |
| 0x003907a0 | 16 | 13 | C_web_connection | {ctor} | __ct__16C_web_connectionFv |
| 0x003907b0 | 356 | GLOBAL | C_web_node | AddConn | AddConn__10C_web_nodeFUsR8S_vectorR8S_vectorUs |
| 0x00390920 | 88 | GLOBAL | C_web_node | DeleteConn | DeleteConn__10C_web_nodeFUs |
| 0x00390980 | 92 | GLOBAL | C_web_node | FindConnIdx | FindConnIdx__10C_web_nodeFUs |
| 0x003909e0 | 60 | GLOBAL | C_web_node | ClearConn | ClearConn__10C_web_nodeFb |
| 0x00390a20 | 512 | GLOBAL | C_web_path | AllocHelpMem | AllocHelpMem__10C_web_pathFPcPc |
| 0x00390c20 | 28 | 13 | C_web_path::S_pathfind | {ctor} | __ct__Q210C_web_path10S_pathfindFv |
| 0x00390c40 | 92 | GLOBAL | C_web_path | FreeHelpMem | FreeHelpMem__10C_web_pathFv |
| 0x00390ca0 | 40 | GLOBAL | C_web_path | GetNextHlpMemBlock | GetNextHlpMemBlock__10C_web_pathFi |
| 0x00390cd0 | 180 | GLOBAL | C_web_path | ClearNode | ClearNode__10C_web_pathFv |
| 0x00390d90 | 56 | GLOBAL | C_web_path | {ctor} | __ct__10C_web_pathFUi |
| 0x00390dd0 | 96 | GLOBAL | C_web_path | {dtor} | __dt__10C_web_pathFv |
| 0x00390e30 | 404 | GLOBAL | C_web_path | Resize | Resize__10C_web_pathFUi |
| 0x00390fd0 | 52 | 13 | C_web_node | {ctor} | __ct__10C_web_nodeFv |
| 0x00391010 | 164 | GLOBAL | C_web_path | AddNode | AddNode__10C_web_pathFfffUs |
| 0x003910c0 | 496 | GLOBAL | C_web_path | DeleteNode | DeleteNode__10C_web_pathFUi |
| 0x003912b0 | 8 | GLOBAL | C_web_path | DrawConnections | DrawConnections__10C_web_pathFv |
| 0x003912c0 | 456 | GLOBAL | C_web_path | CreateConnect | CreateConnect__10C_web_pathFUiUi |
| 0x00391490 | 248 | GLOBAL | C_web_path | LegalConnect | LegalConnect__10C_web_pathFUiUiRUs |
| 0x00391590 | 420 | GLOBAL | C_web_path | ControlNodeConn | ControlNodeConn__10C_web_pathFUi |
| 0x00391740 | 8 | GLOBAL | C_web_path | SelectLine | SelectLine__10C_web_pathFii |
| 0x00391750 | 284 | GLOBAL | C_web_path | SaveNodes | SaveNodes__10C_web_pathFP7C_chunk |
| 0x00391870 | 808 | GLOBAL | C_web_path | LoadNodes | LoadNodes__10C_web_pathFP7C_chunk |
| 0x00391ba0 | 844 | GLOBAL | C_web_path | MergeNodes | MergeNodes__10C_web_pathFP7C_chunk |
| 0x00391ef0 | 448 | GLOBAL | C_web_path | RecompileDistance | RecompileDistance__10C_web_pathFUi |
| 0x003920b0 | 112 | GLOBAL | C_web_path | ReleaseSelect | ReleaseSelect__10C_web_pathFv |
| 0x00392120 | 88 | GLOBAL | C_web_path | FindNodeID | FindNodeID__10C_web_pathFUs |
| 0x00392180 | 1616 | GLOBAL | C_web_path | FindPath_FastByRoads | FindPath_FastByRoads__10C_web_pathFUiUiPQ23std32vector<Ui,Q23std13allocator<Ui>>Us |
| 0x003927d0 | 1828 | GLOBAL | C_web_path | FindPath | FindPath__10C_web_pathFUiUiPQ23std32vector<Ui,Q23std13allocator<Ui>>Us |
| 0x00392f00 | 1580 | GLOBAL | C_web_path | FindPathCompFunc | FindPathCompFunc__10C_web_pathFUiUiPQ23std32vector<Ui,Q23std13allocator<Ui>>PFUsUsUc_fUs |
| 0x00393530 | 1604 | GLOBAL | C_web_path | FindPathCompFunc | FindPathCompFunc__10C_web_pathFUiPFUs_iPQ23std32vector<Ui,Q23std13allocator<Ui>>PFUsUsUc_fUs |
| 0x00393b80 | 172 | GLOBAL | C_web_path | FindSelectedLine | FindSelectedLine__10C_web_pathFRUiRUi |
| 0x00393c30 | 804 | GLOBAL | C_web_path | IncludeNode | IncludeNode__10C_web_pathFUiUi |
| 0x00393f60 | 392 | GLOBAL | C_web_path | FlipOneWay | FlipOneWay__10C_web_pathFUiUib |
| 0x003940f0 | 624 | GLOBAL | C_web_path | GetNearNode | GetNearNode__10C_web_pathFRC8S_vectorfUs |
| 0x00394360 | 424 | GLOBAL | C_web_path | SortNodesByX | SortNodesByX__10C_web_pathFv |
| 0x00394510 | 100 | GLOBAL | C_web_path | IsSorted | IsSorted__10C_web_pathFv |
| 0x00394580 | 144 | GLOBAL | C_web_path | DeRiciConns | DeRiciConns__10C_web_pathFv |
| 0x00394610 | 8 | GLOBAL | C_web_path | DebugFindPath | DebugFindPath__10C_web_pathFUsUs |
| 0x00394620 | 160 | GLOBAL |  | _STDVECTCOPY__FPQ23std32vector<Ui,Q23std13allocator<Ui>>PQ23std32vector<Ui,Q23std13allocator<Ui>> | _STDVECTCOPY__FPQ23std32vector<Ui,Q23std13allocator<Ui>>PQ23std32vector<Ui,Q23std13allocator<Ui>> |
| 0x003946c0 | 916 | GLOBAL |  | nodeproc__FUiUiPFUs_iPFUsUsUc_fPff | nodeproc__FUiUiPFUs_iPFUsUsUc_fPff |
| 0x00394a60 | 152 | GLOBAL | C_web_path | FindPathCompFuncJIPO | FindPathCompFuncJIPO__10C_web_pathFUiPFUs_iPQ23std32vector<Ui,Q23std13allocator<Ui>>PFUsUsUc_fUs |
| 0x00394b00 | 60 | GLOBAL | C_best_time | {ctor} | __ct__11C_best_timeFv |
| 0x00394b40 | 444 | GLOBAL | C_circuit | {ctor} | __ct__9C_circuitFPcPcPUiPUi |
| 0x00394d00 | 324 | GLOBAL | C_circuit | {dtor} | __dt__9C_circuitFv |
| 0x00394e50 | 1188 | GLOBAL | C_circuit | LoadGatesFromScene | LoadGatesFromScene__9C_circuitFv |
| 0x00395300 | 32 | GLOBAL | C_circuit | GetFirstGate | GetFirstGate__9C_circuitFv |
| 0x00395320 | 96 | GLOBAL | C_circuit | ClearGates | ClearGates__9C_circuitFv |
| 0x00395380 | 12 | GLOBAL | C_circuit | ClearPaths | ClearPaths__9C_circuitFv |
| 0x00395390 | 96 | GLOBAL | C_circuit | ClearRaceCams | ClearRaceCams__9C_circuitFv |
| 0x003953f0 | 288 | GLOBAL | C_circuit | LoadPaths | LoadPaths__9C_circuitFUiPQ23std54vector<P10C_web_node,Q23std24allocator<P10C_web_node>> |
| 0x00395510 | 336 | GLOBAL | C_circuit | DetermineRaceCams | DetermineRaceCams__9C_circuitFv |
| 0x00395660 | 208 | GLOBAL | C_circuit | SaveBestTimes | SaveBestTimes__9C_circuitFR8G_Stream |
| 0x00395730 | 344 | GLOBAL | C_circuit | LoadBestTimes | LoadBestTimes__9C_circuitFR8G_Stream |
| 0x00395890 | 344 | GLOBAL | C_circuit | LoadBestTimesDta | LoadBestTimesDta__9C_circuitFi |
| 0x003959f0 | 156 | GLOBAL | C_champ_circuit | {ctor} | __ct__15C_champ_circuitFP9C_circuitUi |
| 0x00395a90 | 156 | GLOBAL | C_champ_circuit | {dtor} | __dt__15C_champ_circuitFv |
| 0x00395b30 | 188 | GLOBAL | C_champ_circuit | LoadProfile | LoadProfile__15C_champ_circuitFR8G_StreamUi |
| 0x00395bf0 | 168 | GLOBAL | C_champ_circuit | SaveProfile | SaveProfile__15C_champ_circuitFR8G_StreamUi |
| 0x00395ca0 | 56 | GLOBAL | C_champ_circuit | SetLockState | SetLockState__15C_champ_circuitF13ER_lock_state |
| 0x00395ce0 | 36 | GLOBAL | C_champ_circuit | SetFinalPos | SetFinalPos__15C_champ_circuitFUiUi |
| 0x00395d10 | 104 | GLOBAL | C_champ_circuit | FinalPositionsClear | FinalPositionsClear__15C_champ_circuitFv |
| 0x00395d80 | 20 | GLOBAL | C_champ_circuit | FinalPositionsRestore | FinalPositionsRestore__15C_champ_circuitFv |
| 0x00395da0 | 20 | GLOBAL | C_champ_circuit | FinalPositionsStore | FinalPositionsStore__15C_champ_circuitFv |
| 0x00395dc0 | 412 | GLOBAL | C_champ_circuit | FinalPositionsSet | FinalPositionsSet__15C_champ_circuitFv |
| 0x00395f60 | 128 | GLOBAL | C_championship | {ctor} | __ct__14C_championshipFUi |
| 0x00395fe0 | 248 | GLOBAL | C_championship | {dtor} | __dt__14C_championshipFv |
| 0x003960e0 | 76 | GLOBAL | C_championship | AddCircuit | AddCircuit__14C_championshipFP15C_champ_circuit |
| 0x00396130 | 160 | GLOBAL | C_championship | LoadProfile | LoadProfile__14C_championshipFR8G_Stream |
| 0x003961d0 | 160 | GLOBAL | C_championship | SaveProfile | SaveProfile__14C_championshipFR8G_Stream |
| 0x00396270 | 192 | GLOBAL | C_championship | SetOpponents | SetOpponents__14C_championshipFPUiPUi |
| 0x00396330 | 392 | GLOBAL | C_racing | FillChampResults | FillChampResults__8C_racingFP14C_championship |
| 0x003964c0 | 284 | GLOBAL | C_opponent | {ctor} | __ct__10C_opponentFv |
| 0x003965e0 | 256 | GLOBAL | C_opponent | {ctor} | __ct__10C_opponentFUi |
| 0x003966e0 | 68 | GLOBAL | C_opponent | {dtor} | __dt__10C_opponentFv |
| 0x00396730 | 196 | GLOBAL | C_opponent | GameDone | GameDone__10C_opponentFv |
| 0x00396800 | 5976 | GLOBAL | C_opponent | GameInit | GameInit__10C_opponentFP17I3D_animation_setUi |
| 0x00397f60 | 172 | GLOBAL | C_opponent | CreateDriver | CreateDriver__10C_opponentFb |
| 0x00398010 | 2784 | GLOBAL | C_opponent | Recover | Recover__10C_opponentFv |
| 0x00398af0 | 348 | GLOBAL | C_opponent | DoRecover | DoRecover__10C_opponentFR8S_vectorR8S_vector |
| 0x00398c50 | 932 | GLOBAL | C_opponent | TickRecover | TickRecover__10C_opponentFUi |
| 0x00399000 | 3716 | GLOBAL | C_opponent | CreateDriverPath | CreateDriverPath__10C_opponentFb |
| 0x00399e90 | 216 | GLOBAL | C_racing | SetRCamIndex | SetRCamIndex__8C_racingFUi |
| 0x00399f70 | 132 | GLOBAL | C_racing | SetRCamIndexFirstRacer | SetRCamIndexFirstRacer__8C_racingFv |
| 0x0039a000 | 136 | GLOBAL | C_opponent | GetRacePoints | GetRacePoints__10C_opponentFv |
| 0x0039a090 | 116 | GLOBAL | C_opponent | GetRacePoints | GetRacePoints__10C_opponentFUi |
| 0x0039a110 | 460 | GLOBAL | C_opponent | SetDeath | SetDeath__10C_opponentFb |
| 0x0039a2e0 | 832 | GLOBAL | C_racing | {ctor} | __ct__8C_racingFv |
| 0x0039a620 | 140 | 13 | std::vector<P17C_racing_osegment,Q23std31allocator<P17C_racing_osegment>> | {dtor} | __dt__Q23std68vector<P17C_racing_osegment,Q23std31allocator<P17C_racing_osegment>>Fv |
| 0x0039a6b0 | 52 | 13 | std::vector<P17C_racing_osegment,Q23std31allocator<P17C_racing_osegment>> | {ctor} | __ct__Q23std68vector<P17C_racing_osegment,Q23std31allocator<P17C_racing_osegment>>Fv |
| 0x0039a6f0 | 348 | GLOBAL | C_racing | {dtor} | __dt__8C_racingFv |
| 0x0039a850 | 372 | GLOBAL | C_racing | LoadCircuits | LoadCircuits__8C_racingFv |
| 0x0039a9d0 | 628 | GLOBAL | C_racing | LoadChamps | LoadChamps__8C_racingFv |
| 0x0039ac50 | 52 | GLOBAL | C_racing | GetCircuit | GetCircuit__8C_racingFUi |
| 0x0039ac90 | 8 | GLOBAL | C_racing | GetCurrentCircuit | GetCurrentCircuit__8C_racingFv |
| 0x0039aca0 | 140 | GLOBAL | C_racing | SetCurrentChamp | SetCurrentChamp__8C_racingFP14C_championship |
| 0x0039ad30 | 144 | GLOBAL | C_racing | SetCurrentChampCircuit | SetCurrentChampCircuit__8C_racingFP15C_champ_circuit |
| 0x0039adc0 | 248 | GLOBAL | C_racing | AddOpponents | AddOpponents__8C_racingFPUiUi |
| 0x0039aec0 | 172 | GLOBAL | C_racing | AddOpponent | AddOpponent__8C_racingFUiUi |
| 0x0039af70 | 984 | GLOBAL | C_racing | GenerateOpponents | GenerateOpponents__8C_racingFUi |
| 0x0039b350 | 120 | GLOBAL | C_racing | ClearOpponents | ClearOpponents__8C_racingFv |
| 0x0039b3d0 | 8 | GLOBAL | C_racing | GetOpponentCount | GetOpponentCount__8C_racingFv |
| 0x0039b3e0 | 236 | GLOBAL | C_racing | GetOpponentName | GetOpponentName__8C_racingFUi |
| 0x0039b4d0 | 516 | GLOBAL | C_racing | GetOpponentNameIdx | GetOpponentNameIdx__8C_racingFPc |
| 0x0039b6e0 | 792 | GLOBAL | C_racing | FillOpponentNameModel | FillOpponentNameModel__8C_racingFP10C_opponentUi |
| 0x0039ba00 | 1424 | GLOBAL | C_racing | PlaceCars | PlaceCars__8C_racingFv |
| 0x0039bf90 | 608 | GLOBAL | C_racing | InitRacers | InitRacers__8C_racingFv |
| 0x0039c1f0 | 184 | GLOBAL |  | compareQtime__FPCvPCv | compareQtime__FPCvPCv |
| 0x0039c2b0 | 152 | GLOBAL |  | compareChampPoint__FPCvPCv | compareChampPoint__FPCvPCv |
| 0x0039c350 | 328 | GLOBAL |  | comparePos__FPCvPCv | comparePos__FPCvPCv |
| 0x0039c4a0 | 352 | GLOBAL | C_racing | LoadChampsProfile | LoadChampsProfile__8C_racingFv |
| 0x0039c600 | 348 | GLOBAL | C_racing | SaveChampsProfile | SaveChampsProfile__8C_racingFv |
| 0x0039c760 | 16 | GLOBAL |  | compSegID__FP11C_best_timeP11C_best_time | compSegID__FP11C_best_timeP11C_best_time |
| 0x0039c770 | 1444 | GLOBAL | C_racing | CreateBestTimes | CreateBestTimes__8C_racingFv |
| 0x0039cd20 | 636 | GLOBAL | C_racing | LoadBestTimes | LoadBestTimes__8C_racingFb |
| 0x0039cfa0 | 404 | GLOBAL | C_racing | SaveBestTimes | SaveBestTimes__8C_racingFb |
| 0x0039d140 | 164 | GLOBAL | C_racing | CanRunRace | CanRunRace__8C_racingFv |
| 0x0039d1f0 | 480 | GLOBAL | C_racing | GameInit | GameInit__8C_racingFv |
| 0x0039d3d0 | 292 | GLOBAL | C_racing | GameDone | GameDone__8C_racingFv |
| 0x0039d500 | 728 | GLOBAL | C_racing | ActivateRacers | ActivateRacers__8C_racingFv |
| 0x0039d7e0 | 188 | GLOBAL | C_opponent | PlayDeath | PlayDeath__10C_opponentFv |
| 0x0039d8a0 | 568 | GLOBAL | C_opponent | Tick | Tick__10C_opponentFUi |
| 0x0039dae0 | 2060 | GLOBAL | C_opponent | TickObjizdecka | TickObjizdecka__10C_opponentFUi |
| 0x0039e2f0 | 724 | GLOBAL | C_opponent | EndOjed | EndOjed__10C_opponentFv |
| 0x0039e5d0 | 8 | GLOBAL | C_opponent | ObjzTestNodeValidCars | ObjzTestNodeValidCars__10C_opponentFR10S_car_node |
| 0x0039e5e0 | 644 | GLOBAL | C_opponent | ObjzTestNodeValid | ObjzTestNodeValid__10C_opponentFR10S_car_nodeUi |
| 0x0039e870 | 784 | GLOBAL | C_opponent | OjedMeZePredu | OjedMeZePredu__10C_opponentFP5C_car |
| 0x0039eb80 | 784 | GLOBAL | C_opponent | OjedMeZleva | OjedMeZleva__10C_opponentFP5C_car |
| 0x0039ee90 | 784 | GLOBAL | C_opponent | OjedMeZprava | OjedMeZprava__10C_opponentFP5C_car |
| 0x0039f1a0 | 784 | GLOBAL | C_opponent | OjedMeZeZadu | OjedMeZeZadu__10C_opponentFP5C_car |
| 0x0039f4b0 | 6168 | GLOBAL | C_opponent | Ojed | Ojed__10C_opponentFP5C_car |
| 0x003a0cd0 | 316 | GLOBAL | C_opponent | FillObjzNode | FillObjzNode__10C_opponentFR8S_vectorfR8S_vectorP10S_car_node |
| 0x003a0e10 | 88 | GLOBAL | C_opponent | GetNextDriverRoute | GetNextDriverRoute__10C_opponentFv |
| 0x003a0e70 | 92 | GLOBAL | C_opponent | GetPrevDriverRoute | GetPrevDriverRoute__10C_opponentFv |
| 0x003a0ed0 | 84 | GLOBAL | C_opponent | GetNextDriverRoute | GetNextDriverRoute__10C_opponentFUi |
| 0x003a0f30 | 92 | GLOBAL | C_opponent | GetPrevDriverRoute | GetPrevDriverRoute__10C_opponentFUi |
| 0x003a0f90 | 2244 | GLOBAL | C_racing | Tick | Tick__8C_racingFUi |
| 0x003a1860 | 324 | GLOBAL | C_opponent | Finish | Finish__10C_opponentFUiUi |
| 0x003a19b0 | 388 | GLOBAL | C_racing | FlushText | FlushText__8C_racingFPcUiUi |
| 0x003a1b40 | 912 | GLOBAL | C_racing | TestBestLap | TestBestLap__8C_racingFP10C_opponentUi |
| 0x003a1ed0 | 528 | GLOBAL | C_racing | TestBestRace | TestBestRace__8C_racingFP10C_opponent |
| 0x003a20e0 | 2560 | GLOBAL | C_racing | EndRace | EndRace__8C_racingFv |
| 0x003a2ae0 | 64 | GLOBAL | C_racing | SetCurrentCircuit | SetCurrentCircuit__8C_racingFP9C_circuit |
| 0x003a2b20 | 412 | GLOBAL | C_racing | GenerateQualification | GenerateQualification__8C_racingFv |
| 0x003a2cc0 | 124 | GLOBAL | C_racing | GetDifficulty2 | GetDifficulty2__8C_racingFv |
| 0x003a2d40 | 128 | GLOBAL | C_racing | GetPowerCoeffByDifficulty | GetPowerCoeffByDifficulty__8C_racingFRfRf |
| 0x003a2dc0 | 20 | GLOBAL | C_racing | GMenuTogleResultsVisibility | GMenuTogleResultsVisibility__8C_racingFv |
| 0x003a2de0 | 8 | GLOBAL | C_racing | GMenuGetResultsVisibility | GMenuGetResultsVisibility__8C_racingFv |
| 0x003a2df0 | 72 | GLOBAL | C_championship | GetResults | GetResults__14C_championshipFP10C_opponent |
| 0x003a2e40 | 60 | GLOBAL | C_championship | GetChampionshipPos | GetChampionshipPos__14C_championshipFP10C_opponent |
| 0x003a2e80 | 340 | GLOBAL | C_racing | LoadReplay | LoadReplay__8C_racingFUiPv |
| 0x003a2fe0 | 80 | GLOBAL | C_racing | GetCurrentCircuitID | GetCurrentCircuitID__8C_racingFv |
| 0x003a3030 | 80 | GLOBAL | C_racing | GetCurrentChampID | GetCurrentChampID__8C_racingFv |
| 0x003a3080 | 40 | GLOBAL |  | GetSegmentIdFromCarIdG__FUi | GetSegmentIdFromCarIdG__FUi |
| 0x003a30b0 | 296 | GLOBAL |  | PrusecikPrimek__FR9S_vector2R9S_vector2R9S_vector2R9S_vector2R9S_vector2RfRf | PrusecikPrimek__FR9S_vector2R9S_vector2R9S_vector2R9S_vector2R9S_vector2RfRf |
| 0x003a31e0 | 372 | GLOBAL |  | PrusecikUsecek__FRC9S_vector2RC9S_vector2RC9S_vector2RC9S_vector2R9S_vector2ffff | PrusecikUsecek__FRC9S_vector2RC9S_vector2RC9S_vector2RC9S_vector2R9S_vector2ffff |
| 0x003a3360 | 8 | GLOBAL | C_racing | SetDamage | SetDamage__8C_racingFb |
| 0x003a3370 | 168 | GLOBAL | C_racing | FakeFinalPositions | FakeFinalPositions__8C_racingFv |
| 0x003a3420 | 300 | GLOBAL | C_racing | TestGate | TestGate__8C_racingFP10C_opponent |
| 0x003a3550 | 164 | GLOBAL | C_racing | TestAllGates | TestAllGates__8C_racingFP10C_opponent |
| 0x003a3600 | 772 | GLOBAL | C_racing | InternTestGate | InternTestGate__8C_racingFP13C_racing_gateR8S_vectorRf |
| 0x003a3910 | 1236 | GLOBAL | C_racing | DoGates | DoGates__8C_racingFP10C_opponent |
| 0x003a3df0 | 72 | GLOBAL | C_racing | IsLastChampCircuit | IsLastChampCircuit__8C_racingFv |
| 0x003a3e40 | 916 | GLOBAL | C_racing | UpdateDriverPowerC | UpdateDriverPowerC__8C_racingFv |
| 0x003a41e0 | 752 | GLOBAL | C_racing | GetDistanceByGates | GetDistanceByGates__8C_racingFP10C_opponentP10C_opponent |
| 0x003a44d0 | 92 | GLOBAL | C_racing | GetOpponentFromCar | GetOpponentFromCar__8C_racingFP5C_car |
| 0x003a4530 | 2520 | GLOBAL | C_racing | TestniPredvidaciKolize | TestniPredvidaciKolize__8C_racingFP5C_carP8C_driver |
| 0x003a4f10 | 2784 | GLOBAL | C_racing | FillDriverCurve | FillDriverCurve__8C_racingFP5C_carP8C_driverPQ23gif49vector<Q211g_collision22gcol_CurveSphereStruct,8>bf |
| 0x003a59f0 | 104 | GLOBAL | C_racing | GetRecoverTime | GetRecoverTime__8C_racingFv |
| 0x003a5a60 | 120 | GLOBAL | C_championship | SaveBestTimes | SaveBestTimes__14C_championshipFR8G_Stream |
| 0x003a5ae0 | 120 | GLOBAL | C_championship | LoadBestTimes | LoadBestTimes__14C_championshipFR8G_Stream |
| 0x003a5b60 | 120 | GLOBAL | C_championship | LoadBestTimesDta | LoadBestTimesDta__14C_championshipFi |
| 0x003a5be0 | 208 | GLOBAL | C_champ_circuit | SaveBestTimes | SaveBestTimes__15C_champ_circuitFR8G_Stream |
| 0x003a5cb0 | 344 | GLOBAL | C_champ_circuit | LoadBestTimes | LoadBestTimes__15C_champ_circuitFR8G_Stream |
| 0x003a5e10 | 344 | GLOBAL | C_champ_circuit | LoadBestTimesDta | LoadBestTimesDta__15C_champ_circuitFi |
| 0x003a5f70 | 1356 | GLOBAL | C_racing | PreloadWrecks | PreloadWrecks__8C_racingFv |
| 0x003a64c0 | 916 | 13 | std | sort<PP11C_best_time,PFP11C_best_timeP11C_best_time_b> | sort<PP11C_best_time,PFP11C_best_timeP11C_best_time_b>__3stdFPP11C_best_timePP11C_best_timePFP11C_best_timeP11C_best_time_b |
| 0x003a6860 | 252 | 13 |  | __sort132<PP11C_best_time,PFP11C_best_timeP11C_best_time_b>__3stdFPP11C_best_timePP11C_best_timePP11C_best_timePFP11C_best_timeP11C_best_time_b | __sort132<PP11C_best_time,PFP11C_best_timeP11C_best_time_b>__3stdFPP11C_best_timePP11C_best_timePP11C_best_timePFP11C_best_timeP11C_best_time_b |
| 0x003a6960 | 204 | 13 |  | __selection_sort<PP11C_best_time,PFP11C_best_timeP11C_best_time_b>__3stdFPP11C_best_timePP11C_best_timePFP11C_best_timeP11C_best_time_b | __selection_sort<PP11C_best_time,PFP11C_best_timeP11C_best_time_b>__3stdFPP11C_best_timePP11C_best_timePFP11C_best_timeP11C_best_time_b |
| 0x003a6a30 | 1728 | GLOBAL |  | Compress__FPvUiPPvPUib | Compress__FPvUiPPvPUib |
| 0x003a70f0 | 44 | GLOBAL | C_roads | {ctor} | __ct__7C_roadsFv |
| 0x003a7120 | 76 | GLOBAL | C_roads | {dtor} | __dt__7C_roadsFv |
| 0x003a7170 | 1932 | GLOBAL | C_roads | SaveData | SaveData__7C_roadsFP7C_chunk |
| 0x003a7900 | 2676 | GLOBAL | C_roads | LoadData | LoadData__7C_roadsFP7C_chunk |
| 0x003a8380 | 8 | 13 | std::__vector_deleter<13S_road_node_1,Q23std26allocator<13S_road_node_1>> | data | data__Q23std68__vector_deleter<13S_road_node_1,Q23std26allocator<13S_road_node_1>>Fv |
| 0x003a8390 | 8 | 13 | S_road_node | {ctor} | __ct__11S_road_nodeFv |
| 0x003a83a0 | 8 | 13 | std::__vector_deleter<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>> | data | data__Q23std76__vector_deleter<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>>Fv |
| 0x003a83b0 | 8 | 13 | S_road_crossing | {ctor} | __ct__15S_road_crossingFv |
| 0x003a83c0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_road_node_1>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_road_node_1>,Ui,1>FUi |
| 0x003a83d0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std30allocator<17S_road_crossing_1>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details64compressed_pair_imp<Q23std30allocator<17S_road_crossing_1>,Ui,1>FUi |
| 0x003a83e0 | 7852 | GLOBAL | C_roads | RecompileDistances | RecompileDistances__7C_roadsFv |
| 0x003aa290 | 504 | GLOBAL | C_roads | GetNearNode | GetNearNode__7C_roadsFRC8S_vector |
| 0x003aa490 | 168 | GLOBAL | C_roads | FindCrossing | FindCrossing__7C_roadsFUsUs |
| 0x003aa540 | 364 | GLOBAL | C_roads | NPC_FindCrossIdxToCross | NPC_FindCrossIdxToCross__7C_roadsFUsUsUs |
| 0x003aa6b0 | 1932 | GLOBAL | C_roads | NPC_FindPathFromNodeToCross | NPC_FindPathFromNodeToCross__7C_roadsFUsR8S_vectorPQ23gif22vector<10S_car_node,8> |
| 0x003aae40 | 692 | GLOBAL | C_roads | NPC_FindPathToNextCross1 | NPC_FindPathToNextCross1__7C_roadsFUsiPQ23gif22vector<10S_car_node,8> |
| 0x003ab100 | 288 | GLOBAL | C_roads | FillPathFromNodeToCross | FillPathFromNodeToCross__7C_roadsFUsUsRQ23std32vector<Us,Q23std13allocator<Us>>UsUs |
| 0x003ab220 | 372 | GLOBAL | C_roads | FindCrossingFromNode | FindCrossingFromNode__7C_roadsFUsUsRf |
| 0x003ab3a0 | 424 | GLOBAL | C_roads | FindNearCrossingFromNode | FindNearCrossingFromNode__7C_roadsFUsRib |
| 0x003ab550 | 1268 | GLOBAL | C_roads | FindPathBeweenCross | FindPathBeweenCross__7C_roadsFUsUsRQ23std32vector<Us,Q23std13allocator<Us>> |
| 0x003aba50 | 28 | 13 | C_roads::S_pathfind$3865road_web_cpp | {ctor} | __ct__Q27C_roads27S_pathfind$3865road_web_cppFv |
| 0x003aba70 | 244 | GLOBAL | C_roads | FillPathFromCrossToCross | FillPathFromCrossToCross__7C_roadsFUsUsRQ23std32vector<Us,Q23std13allocator<Us>>UsUs |
| 0x003abb70 | 1168 | GLOBAL | C_roads | FindPath | FindPath__7C_roadsFUsUsRQ23std32vector<Us,Q23std13allocator<Us>> |
| 0x003ac000 | 600 | GLOBAL | C_roads | SortNodesByX | SortNodesByX__7C_roadsFv |
| 0x003ac260 | 100 | GLOBAL | C_roads | IsSorted | IsSorted__7C_roadsFv |
| 0x003ac2d0 | 208 | GLOBAL | C_roads | DestroyAllOffPaths | DestroyAllOffPaths__7C_roadsFv |
| 0x003ac3a0 | 148 | WEAK | std::__vector_imp<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>,0> | resize | resize__Q23std74__vector_imp<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>,0>FUiRC17S_road_crossing_1 |
| 0x003ac440 | 124 | WEAK | std::__vector_imp<13S_road_node_1,Q23std26allocator<13S_road_node_1>,0> | resize | resize__Q23std66__vector_imp<13S_road_node_1,Q23std26allocator<13S_road_node_1>,0>FUiRC13S_road_node_1 |
| 0x003ac4c0 | 104 | WEAK | std::__vector_deleter<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>> | {dtor} | __dt__Q23std76__vector_deleter<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>>Fv |
| 0x003ac530 | 104 | WEAK | std::__vector_deleter<13S_road_node_1,Q23std26allocator<13S_road_node_1>> | {dtor} | __dt__Q23std68__vector_deleter<13S_road_node_1,Q23std26allocator<13S_road_node_1>>Fv |
| 0x003ac5a0 | 368 | WEAK | std::__vector_imp<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>,0> | erase | erase__Q23std74__vector_imp<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>,0>FP17S_road_crossing_1P17S_road_crossing_1 |
| 0x003ac710 | 1500 | WEAK | std::__vector_imp<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>,0> | insert | insert__Q23std74__vector_imp<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>,0>FP17S_road_crossing_1UiRC17S_road_crossing_1 |
| 0x003accf0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std30allocator<17S_road_crossing_1>,Ui,1> | swap | swap__Q310Metrowerks7details64compressed_pair_imp<Q23std30allocator<17S_road_crossing_1>,Ui,1>FRQ310Metrowerks7details64compressed_pair_imp<Q23std30allocator<17S_road_crossing_1>,Ui,1> |
| 0x003acd10 | 304 | WEAK | std::__vector_imp<13S_road_node_1,Q23std26allocator<13S_road_node_1>,0> | erase | erase__Q23std66__vector_imp<13S_road_node_1,Q23std26allocator<13S_road_node_1>,0>FP13S_road_node_1P13S_road_node_1 |
| 0x003ace40 | 1396 | WEAK | std::__vector_imp<13S_road_node_1,Q23std26allocator<13S_road_node_1>,0> | insert | insert__Q23std66__vector_imp<13S_road_node_1,Q23std26allocator<13S_road_node_1>,0>FP13S_road_node_1UiRC13S_road_node_1 |
| 0x003ad3c0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_road_node_1>,Ui,1> | swap | swap__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_road_node_1>,Ui,1>FRQ310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_road_node_1>,Ui,1> |
| 0x003ad3e0 | 88 | WEAK | std::__vector_deleter<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>> | clear | clear__Q23std76__vector_deleter<17S_road_crossing_1,Q23std30allocator<17S_road_crossing_1>>Fv |
| 0x003ad440 | 80 | WEAK | std::__vector_deleter<13S_road_node_1,Q23std26allocator<13S_road_node_1>> | clear | clear__Q23std68__vector_deleter<13S_road_node_1,Q23std26allocator<13S_road_node_1>>Fv |
| 0x003ad490 | 288 | GLOBAL | C_compass | Activate | Activate__9C_compassFP9I3D_frame |
| 0x003ad5b0 | 180 | GLOBAL | C_silver_screen | Tick | Tick__15C_silver_screenFUi |
| 0x003ad670 | 20 | GLOBAL | C_fire | {ctor} | __ct__6C_fireFv |
| 0x003ad690 | 152 | GLOBAL | C_fire | {dtor} | __dt__6C_fireFv |
| 0x003ad730 | 652 | GLOBAL | C_fire | Init | Init__6C_fireFR8S_vectorP9I3D_frameiffb |
| 0x003ad9c0 | 1076 | GLOBAL | C_fire | Tick | Tick__6C_fireFUi |
| 0x003ade00 | 1952 | GLOBAL | C_using_object | AddObject | AddObject__14C_using_objectFPQ214C_using_object14S_using_object |
| 0x003ae5a0 | 348 | GLOBAL | C_using_object | DelObject | DelObject__14C_using_objectFPQ214C_using_object14S_using_object |
| 0x003ae700 | 308 | GLOBAL | C_using_object | ClearObjects | ClearObjects__14C_using_objectFv |
| 0x003ae840 | 664 | GLOBAL | C_using_object | Tick | Tick__14C_using_objectFUi |
| 0x003aeae0 | 364 | GLOBAL | C_using_object | IsAnyObjectNear | IsAnyObjectNear__14C_using_objectFRC8S_vectorRC8S_vectorP9I3D_frame |
| 0x003aec50 | 1192 | GLOBAL | C_using_object | FindNearObjects | FindNearObjects__14C_using_objectFRC8S_vectorRC8S_vectorRQ23gif22vector<10S_GameItem,8>P9I3D_frame |
| 0x003af100 | 844 | GLOBAL | C_using_object | FindAutoPickObjects | FindAutoPickObjects__14C_using_objectFRC8S_vectorRC8S_vectorRQ23gif22vector<10S_GameItem,8> |
| 0x003af450 | 8 | GLOBAL | C_using_object | ClearInvalidItems | ClearInvalidItems__14C_using_objectFv |
| 0x003af460 | 136 | GLOBAL | C_using_object | AddInvalidItem | AddInvalidItem__14C_using_objectFUi |
| 0x003af4f0 | 96 | GLOBAL | C_using_object | DelInvalidItem | DelInvalidItem__14C_using_objectFUi |
| 0x003af550 | 76 | GLOBAL | C_using_object | IsInvalidItem | IsInvalidItem__14C_using_objectFUi |
| 0x003af5a0 | 248 | GLOBAL | C_using_object | SaveGameGetSize | SaveGameGetSize__14C_using_objectFv |
| 0x003af6a0 | 984 | GLOBAL | C_using_object | SaveGameSave | SaveGameSave__14C_using_objectFPPv |
| 0x003afa80 | 1888 | GLOBAL | C_using_object | SaveGameLoad | SaveGameLoad__14C_using_objectFPPv |
| 0x003b01e0 | 88 | GLOBAL | C_sound_table | {ctor} | __ct__13C_sound_tableFv |
| 0x003b0240 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std43allocator<Q213C_sound_table13S_sound_table>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details77compressed_pair_imp<Q23std43allocator<Q213C_sound_table13S_sound_table>,Ui,1>FUi |
| 0x003b0250 | 120 | GLOBAL | C_sound_table | {dtor} | __dt__13C_sound_tableFv |
| 0x003b02d0 | 216 | GLOBAL | C_sound_table | FreeTables | FreeTables__13C_sound_tableFv |
| 0x003b03b0 | 188 | GLOBAL | C_sound_table | AddGroup | AddGroup__13C_sound_tableFPCc |
| 0x003b0470 | 356 | GLOBAL | C_sound_table | DelGroup | DelGroup__13C_sound_tableFi |
| 0x003b05e0 | 8 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_sound_group>,Ui,1> | first | first__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_sound_group>,Ui,1>Fv |
| 0x003b05f0 | 572 | GLOBAL | C_sound_table | AddSound | AddSound__13C_sound_tableFPCcUi |
| 0x003b0830 | 240 | GLOBAL | C_sound_table | DelSound | DelSound__13C_sound_tableFUi |
| 0x003b0920 | 92 | GLOBAL | C_sound_table | GetTableFromIDX | GetTableFromIDX__13C_sound_tableFUi |
| 0x003b0980 | 104 | GLOBAL | C_sound_table | SetFileName | SetFileName__13C_sound_tableFPQ213C_sound_table13S_sound_tableiPCc |
| 0x003b09f0 | 172 | GLOBAL | C_sound_table | OptimalizeTable | OptimalizeTable__13C_sound_tableFPQ213C_sound_table13S_sound_table |
| 0x003b0aa0 | 1252 | GLOBAL | C_sound_table | LoadTables | LoadTables__13C_sound_tableFPCc |
| 0x003b0f90 | 652 | GLOBAL | C_sound_table | SaveTables | SaveTables__13C_sound_tableFPCc |
| 0x003b1220 | 1840 | GLOBAL | C_sound_table | ConstructSound | ConstructSound__13C_sound_tableFP9I3D_soundUi |
| 0x003b1950 | 200 | GLOBAL | C_sound_table | GameInit | GameInit__13C_sound_tableFv |
| 0x003b1a20 | 200 | GLOBAL | C_sound_table | GameDone | GameDone__13C_sound_tableFv |
| 0x003b1af0 | 1476 | GLOBAL | C_sound_table | PreloadSoundPermanent | PreloadSoundPermanent__13C_sound_tableFUi |
| 0x003b20c0 | 612 | GLOBAL | C_sound_table | GetPreloadedSound | GetPreloadedSound__13C_sound_tableFUi |
| 0x003b2330 | 8 | GLOBAL | C_sound_table | TestConstructSound | TestConstructSound__13C_sound_tableFP9I3D_soundUi |
| 0x003b2340 | 72 | GLOBAL | C_sound_table | ClearInvisibleFlag | ClearInvisibleFlag__13C_sound_tableFv |
| 0x003b2390 | 436 | GLOBAL | soundArena | SoundEmit | SoundEmit__10soundArenaFR8S_vectorffUiP7C_actor |
| 0x003b2550 | 560 | GLOBAL | soundArena | GetSounds | GetSounds__10soundArenaFP8C_entity |
| 0x003b2780 | 52 | GLOBAL | soundArena | Init | Init__10soundArenaFv |
| 0x003b27c0 | 56 | GLOBAL | soundArena | InvalidateActor | InvalidateActor__10soundArenaFP7C_actor |
| 0x003b2800 | 44 | GLOBAL | S_auto_hide_machine | {ctor} | __ct__19S_auto_hide_machineFv |
| 0x003b2830 | 124 | GLOBAL | S_auto_hide_machine | {dtor} | __dt__19S_auto_hide_machineFv |
| 0x003b28b0 | 128 | GLOBAL | S_auto_hide_machine | Init | Init__19S_auto_hide_machineFv |
| 0x003b2930 | 264 | GLOBAL | S_auto_hide_machine | Done | Done__19S_auto_hide_machineFv |
| 0x003b2a40 | 436 | GLOBAL | S_auto_hide_machine | Tick | Tick__19S_auto_hide_machineFUi |
| 0x003b2c00 | 188 | GLOBAL | S_auto_hide_machine | ProcessItem | ProcessItem__19S_auto_hide_machineFPQ219S_auto_hide_machine6S_itemb |
| 0x003b2cc0 | 600 | GLOBAL | S_auto_hide_machine | AddArea | AddArea__19S_auto_hide_machineFP8I3D_area |
| 0x003b2f20 | 580 | 13 | gif::vector<P9I3D_frame,8> | {ctor} | __ct__Q23gif21vector<P9I3D_frame,8>FRCQ23gif21vector<P9I3D_frame,8> |
| 0x003b3170 | 260 | 13 | gif::vector<P9I3D_frame,8> | reserve | reserve__Q23gif21vector<P9I3D_frame,8>Fi |
| 0x003b3280 | 396 | 13 | gif::vector<Q219S_auto_hide_machine6S_item,8> | reserve | reserve__Q23gif40vector<Q219S_auto_hide_machine6S_item,8>Fi |
| 0x003b3410 | 256 | GLOBAL | S_auto_hide_machine | DelArea | DelArea__19S_auto_hide_machineFP8I3D_area |
| 0x003b3510 | 540 | 13 | gif::vector<P9I3D_frame,8> | operator= | __as__Q23gif21vector<P9I3D_frame,8>FRCQ23gif21vector<P9I3D_frame,8> |
| 0x003b3730 | 508 | GLOBAL | S_auto_hide_machine | AddFrameToArea | AddFrameToArea__19S_auto_hide_machineFP8I3D_areaP9I3D_frame |
| 0x003b3930 | 336 | GLOBAL | S_auto_hide_machine | DelFrameFromArea | DelFrameFromArea__19S_auto_hide_machineFP8I3D_areaP9I3D_frame |
| 0x003b3a80 | 328 | GLOBAL | S_auto_hide_machine | ClearAll | ClearAll__19S_auto_hide_machineFv |
| 0x003b3bd0 | 172 | GLOBAL | S_auto_hide_machine | GetNumFrames | GetNumFrames__19S_auto_hide_machineFP8I3D_area |
| 0x003b3c80 | 140 | GLOBAL | S_auto_hide_machine | GetFrames | GetFrames__19S_auto_hide_machineFP8I3D_area |
| 0x003b3d10 | 160 | GLOBAL | S_auto_hide_machine | SetAreaPeriod | SetAreaPeriod__19S_auto_hide_machineFP8I3D_areai |
| 0x003b3db0 | 140 | GLOBAL | S_auto_hide_machine | GetAreaPeriod | GetAreaPeriod__19S_auto_hide_machineFP8I3D_area |
| 0x003b3e40 | 136 | GLOBAL | S_auto_hide_machine | SetAreaOnlyIn | SetAreaOnlyIn__19S_auto_hide_machineFP8I3D_areab |
| 0x003b3ed0 | 140 | GLOBAL | S_auto_hide_machine | GetAreaOnlyIn | GetAreaOnlyIn__19S_auto_hide_machineFP8I3D_area |
| 0x003b3f60 | 240 | GLOBAL | S_auto_hide_machine | DelFrameFromAllAreas | DelFrameFromAllAreas__19S_auto_hide_machineFP9I3D_frame |
| 0x003b4050 | 8 | GLOBAL | S_auto_hide_machine | OptimalizeAll | OptimalizeAll__19S_auto_hide_machineFv |
| 0x003b4060 | 1376 | GLOBAL | S_auto_hide_machine | SaveData | SaveData__19S_auto_hide_machineFP7C_chunk |
| 0x003b45c0 | 2296 | GLOBAL | S_auto_hide_machine | LoadData | LoadData__19S_auto_hide_machineFP7C_chunk |
| 0x003b4ec0 | 52 | 13 | S_auto_hide_machine | GetNumAreas | GetNumAreas__19S_auto_hide_machineFv |
| 0x003b4f00 | 8 | 13 | S_auto_hide_machine | GetAreas | GetAreas__19S_auto_hide_machineFv |
| 0x003b4f10 | 8 | 13 | C_sound_table | GetNumSounds | GetNumSounds__13C_sound_tableFv |
| 0x003b4f20 | 8 | 13 | C_sound_table | GetSounds | GetSounds__13C_sound_tableFv |
| 0x003b4f30 | 80 | WEAK | std::__vector_deleter<13S_sound_group,Q23std26allocator<13S_sound_group>> | clear | clear__Q23std68__vector_deleter<13S_sound_group,Q23std26allocator<13S_sound_group>>Fv |
| 0x003b4f80 | 1516 | WEAK | std::__vector_imp<13S_sound_group,Q23std26allocator<13S_sound_group>,0> | insert | insert__Q23std66__vector_imp<13S_sound_group,Q23std26allocator<13S_sound_group>,0>FP13S_sound_groupUiRC13S_sound_group |
| 0x003b5570 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_sound_group>,Ui,1> | swap | swap__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_sound_group>,Ui,1>FRQ310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_sound_group>,Ui,1> |
| 0x003b5590 | 216 | WEAK | std::__vector_imp<Q213C_sound_table13S_sound_table,Q23std43allocator<Q213C_sound_table13S_sound_table>,0> | erase | erase__Q23std100__vector_imp<Q213C_sound_table13S_sound_table,Q23std43allocator<Q213C_sound_table13S_sound_table>,0>FPQ213C_sound_table13S_sound_table |
| 0x003b5670 | 244 | 13 | C_sound_table::S_sound_table | operator= | __as__Q213C_sound_table13S_sound_tableFRCQ213C_sound_table13S_sound_table |
| 0x003b5770 | 1292 | WEAK | std::__vector_imp<Q213C_sound_table13S_sound_table,Q23std43allocator<Q213C_sound_table13S_sound_table>,0> | insert | insert__Q23std100__vector_imp<Q213C_sound_table13S_sound_table,Q23std43allocator<Q213C_sound_table13S_sound_table>,0>FPQ213C_sound_table13S_sound_tableUiRCQ213C_sound_table13S_sound_table |
| 0x003b5c80 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std43allocator<Q213C_sound_table13S_sound_table>,Ui,1> | swap | swap__Q310Metrowerks7details77compressed_pair_imp<Q23std43allocator<Q213C_sound_table13S_sound_table>,Ui,1>FRQ310Metrowerks7details77compressed_pair_imp<Q23std43allocator<Q213C_sound_table13S_sound_table>,Ui,1> |
| 0x003b5ca0 | 88 | WEAK | std::__vector_deleter<Q213C_sound_table13S_sound_table,Q23std43allocator<Q213C_sound_table13S_sound_table>> | clear | clear__Q23std102__vector_deleter<Q213C_sound_table13S_sound_table,Q23std43allocator<Q213C_sound_table13S_sound_table>>Fv |
| 0x003b5d00 | 104 | WEAK | std::__vector_deleter<Q213C_sound_table13S_sound_table,Q23std43allocator<Q213C_sound_table13S_sound_table>> | {dtor} | __dt__Q23std102__vector_deleter<Q213C_sound_table13S_sound_table,Q23std43allocator<Q213C_sound_table13S_sound_table>>Fv |
| 0x003b5d70 | 104 | WEAK | std::__vector_deleter<13S_sound_group,Q23std26allocator<13S_sound_group>> | {dtor} | __dt__Q23std68__vector_deleter<13S_sound_group,Q23std26allocator<13S_sound_group>>Fv |
| 0x003b5de0 | 92 | 13 | std::vector<13S_sound_group,Q23std26allocator<13S_sound_group>> | {dtor} | __dt__Q23std58vector<13S_sound_group,Q23std26allocator<13S_sound_group>>Fv |
| 0x003b5e40 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_sound_group>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_sound_group>,Ui,1>FUi |
| 0x003b5e50 | 760 | GLOBAL | S_tshift_elem | Tick | Tick__13S_tshift_elemFf |
| 0x003b6150 | 52 | GLOBAL | C_tshift | {ctor} | __ct__8C_tshiftFv |
| 0x003b6190 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std26allocator<13S_tshift_elem>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details60compressed_pair_imp<Q23std26allocator<13S_tshift_elem>,Ui,1>FUi |
| 0x003b61a0 | 100 | GLOBAL | C_tshift | {dtor} | __dt__8C_tshiftFv |
| 0x003b6210 | 168 | GLOBAL | C_tshift | Tick | Tick__8C_tshiftFUi |
| 0x003b62c0 | 3612 | GLOBAL |  | PrepareGlassSingleObject__FR6S_DICE | PrepareGlassSingleObject__FR6S_DICE |
| 0x003b70e0 | 144 | GLOBAL | C_Break_plain_glass | Init | Init__19C_Break_plain_glassFi |
| 0x003b7170 | 1096 | GLOBAL | C_Break_plain_glass | Done | Done__19C_Break_plain_glassFv |
| 0x003b75c0 | 5144 | GLOBAL | C_Break_plain_glass | Tick | Tick__19C_Break_plain_glassFUi |
| 0x003b89e0 | 2140 | GLOBAL | C_Break_plain_glass | BreakGlass | BreakGlass__19C_Break_plain_glassFP9I3D_framePC8S_vectorPCcb |
| 0x003b9240 | 284 | 13 | gif::vector<6S_DICE,8> | reserve | reserve__Q23gif17vector<6S_DICE,8>Fi |
| 0x003b9360 | 8 | 13 | S_DICE_OBJ | {ctor} | __ct__10S_DICE_OBJFv |
| 0x003b9370 | 7012 | GLOBAL | C_Break_plain_glass | CutMesh | CutMesh__19C_Break_plain_glassFP9I3D_framePQ23gif23vector<11I3D_triface,8>PQ23gif22vector<10I3D_vertex,8>P12I3D_materialR8S_matrix |
| 0x003baee0 | 292 | 13 | gif::vector<10I3D_vertex,8> | reserve | reserve__Q23gif22vector<10I3D_vertex,8>Fi |
| 0x003bb010 | 352 | 13 | gif::vector<11I3D_triface,8> | reserve | reserve__Q23gif23vector<11I3D_triface,8>Fi |
| 0x003bb170 | 2048 | GLOBAL | C_Break_plain_glass | PrepareObject | PrepareObject__19C_Break_plain_glassFiPQ23gif23vector<11I3D_triface,8>PQ23gif22vector<10I3D_vertex,8>P12I3D_materialP10I3D_object |
| 0x003bb970 | 208 | GLOBAL | C_Debris | Init | Init__8C_DebrisFf |
| 0x003bba40 | 320 | GLOBAL | C_Debris | Done | Done__8C_DebrisFv |
| 0x003bbb80 | 1424 | GLOBAL | C_Debris | Tick | Tick__8C_DebrisFUi |
| 0x003bc110 | 1556 | GLOBAL | C_Debris | GenerateDebris | GenerateDebris__8C_DebrisFRC8S_vectorifffPC8S_vector |
| 0x003bc730 | 72 | 13 | S_DEBRIS | {ctor} | __ct__8S_DEBRISFRC8S_DEBRIS |
| 0x003bc780 | 316 | 13 | gif::vector<8S_DEBRIS,8> | reserve | reserve__Q23gif19vector<8S_DEBRIS,8>Fi |
| 0x003bc8c0 | 2084 | GLOBAL | C_Debris | GenerateFlash | GenerateFlash__8C_DebrisFP12S_FLASH_INIT |
| 0x003bd0f0 | 64 | 13 | S_FLASH | {ctor} | __ct__7S_FLASHFRC7S_FLASH |
| 0x003bd130 | 380 | 13 | gif::vector<7S_FLASH,8> | reserve | reserve__Q23gif18vector<7S_FLASH,8>Fi |
| 0x003bd2b0 | 1080 | GLOBAL | C_Debris | PrepareObject | PrepareObject__8C_DebrisFP10I3D_objectfPC8S_vector |
| 0x003bd6f0 | 8 | GLOBAL |  | __LSI_blueassert__FPCciPCc | __LSI_blueassert__FPCciPCc |
| 0x003bd6f8 | 400 | GLOBAL |  | sceGsResetGraph | sceGsResetGraph |
| 0x003bd888 | 12 | GLOBAL |  | sceGsGetGParam | sceGsGetGParam |
| 0x003bd898 | 104 | GLOBAL |  | sceGsResetPath | sceGsResetPath |
| 0x003bd900 | 160 | LOCAL |  | checkModelVersion | checkModelVersion |
| 0x003bd9a0 | 16 | LOCAL |  | _GetGsDxDyOffset | _GetGsDxDyOffset |
| 0x003bd9b0 | 884 | GLOBAL |  | sceGsSetDefDispEnv | sceGsSetDefDispEnv |
| 0x003bdd28 | 188 | GLOBAL |  | sceGsPutDispEnv | sceGsPutDispEnv |
| 0x003bdde8 | 200 | GLOBAL |  | sceGszbufaddr | sceGszbufaddr |
| 0x003bdeb0 | 484 | GLOBAL |  | sceGsSetDefDrawEnv | sceGsSetDefDrawEnv |
| 0x003be098 | 260 | GLOBAL |  | sceGsSetDefClear | sceGsSetDefClear |
| 0x003be1a0 | 232 | GLOBAL |  | sceGsPutDrawEnv | sceGsPutDrawEnv |
| 0x003be288 | 664 | GLOBAL |  | sceGsSetDefDBuff | sceGsSetDefDBuff |
| 0x003be520 | 92 | GLOBAL |  | sceGsSwapDBuff | sceGsSwapDBuff |
| 0x003be580 | 148 | GLOBAL |  | sceGsSyncV | sceGsSyncV |
| 0x003be618 | 788 | GLOBAL |  | sceGsSyncPath | sceGsSyncPath |
| 0x003be930 | 484 | GLOBAL |  | sceGsSetDefLoadImage | sceGsSetDefLoadImage |
| 0x003beb18 | 380 | GLOBAL |  | sceGsExecLoadImage | sceGsExecLoadImage |
| 0x003bec98 | 112 | LOCAL |  | WaitDma | WaitDma |
| 0x003bed08 | 132 | LOCAL |  | WatchDma | WatchDma |
| 0x003bed90 | 40 | LOCAL |  | CheckAddress | CheckAddress |
| 0x003bedb8 | 52 | LOCAL |  | memclr | memclr |
| 0x003bedf0 | 40 | GLOBAL |  | sceDmaGetChan | sceDmaGetChan |
| 0x003bee18 | 220 | GLOBAL |  | sceDmaReset | sceDmaReset |
| 0x003beef8 | 472 | GLOBAL |  | sceDmaPutEnv | sceDmaPutEnv |
| 0x003bf0d0 | 56 | GLOBAL |  | sceDmaGetEnv | sceDmaGetEnv |
| 0x003bf108 | 20 | GLOBAL |  | sceDmaPutStallAddr | sceDmaPutStallAddr |
| 0x003bf120 | 104 | GLOBAL |  | sceDmaSend | sceDmaSend |
| 0x003bf188 | 116 | GLOBAL |  | sceDmaSendN | sceDmaSendN |
| 0x003bf200 | 116 | GLOBAL |  | sceDmaSendI | sceDmaSendI |
| 0x003bf278 | 72 | GLOBAL |  | sceDmaRecv | sceDmaRecv |
| 0x003bf2c0 | 124 | GLOBAL |  | sceDmaRecvN | sceDmaRecvN |
| 0x003bf340 | 128 | GLOBAL |  | sceDmaRecvI | sceDmaRecvI |
| 0x003bf3c0 | 56 | GLOBAL |  | sceDmaSync | sceDmaSync |
| 0x003bf3f8 | 52 | GLOBAL |  | sceDmaWatch | sceDmaWatch |
| 0x003bf430 | 152 | GLOBAL |  | sceDmaPause | sceDmaPause |
| 0x003bf4c8 | 20 | GLOBAL |  | sceDmaRestart | sceDmaRestart |
| 0x003bf4e0 | 44 | LOCAL |  | sceVuCheckBusy | sceVuCheckBusy |
| 0x003bf510 | 64 | LOCAL |  | sceVu0MemWriteQ | sceVu0MemWriteQ |
| 0x003bf550 | 68 | LOCAL |  | sceVu0MemReadQ | sceVu0MemReadQ |
| 0x003bf598 | 20 | GLOBAL |  | sceDevVu1Reset | sceDevVu1Reset |
| 0x003bf5b0 | 96 | GLOBAL |  | sceDevVu1Pause | sceDevVu1Pause |
| 0x003bf610 | 80 | GLOBAL |  | sceDevVu1Continue | sceDevVu1Continue |
| 0x003bf660 | 48 | GLOBAL |  | sceDevVu1PutDBit | sceDevVu1PutDBit |
| 0x003bf690 | 48 | GLOBAL |  | sceDevVu1PutTBit | sceDevVu1PutTBit |
| 0x003bf6c0 | 16 | GLOBAL |  | sceDevVu1GetDBit | sceDevVu1GetDBit |
| 0x003bf6d0 | 16 | GLOBAL |  | sceDevVu1GetTBit | sceDevVu1GetTBit |
| 0x003bf6e0 | 72 | LOCAL |  | sceDevVu1Sync | sceDevVu1Sync |
| 0x003bf728 | 16 | GLOBAL |  | sceDevVu1Exec | sceDevVu1Exec |
| 0x003bf738 | 36 | GLOBAL |  | sceDevVu1GetTpc | sceDevVu1GetTpc |
| 0x003bf760 | 396 | GLOBAL |  | sceDevVu1GetCnd | sceDevVu1GetCnd |
| 0x003bf8f0 | 328 | GLOBAL |  | sceDevVu1PutCnd | sceDevVu1PutCnd |
| 0x003bfa38 | 16 | GLOBAL |  | sceGifPkInit | sceGifPkInit |
| 0x003bfa48 | 16 | GLOBAL |  | sceGifPkReset | sceGifPkReset |
| 0x003bfa58 | 88 | GLOBAL |  | sceGifPkTerminate | sceGifPkTerminate |
| 0x003bfab0 | 20 | GLOBAL |  | sceGifPkSize | sceGifPkSize |
| 0x003bfac8 | 116 | GLOBAL |  | sceGifPkCnt | sceGifPkCnt |
| 0x003bfb40 | 152 | GLOBAL |  | sceGifPkRef | sceGifPkRef |
| 0x003bfbd8 | 140 | GLOBAL |  | sceGifPkCall | sceGifPkCall |
| 0x003bfc68 | 116 | GLOBAL |  | sceGifPkRet | sceGifPkRet |
| 0x003bfce0 | 116 | GLOBAL |  | sceGifPkEnd | sceGifPkEnd |
| 0x003bfd58 | 40 | GLOBAL |  | sceGifPkAddGsData | sceGifPkAddGsData |
| 0x003bfd80 | 16 | GLOBAL |  | sceVif1PkInit | sceVif1PkInit |
| 0x003bfd90 | 88 | GLOBAL |  | sceVif1PkTerminate | sceVif1PkTerminate |
| 0x003bfde8 | 20 | GLOBAL |  | sceVif1PkSize | sceVif1PkSize |
| 0x003bfe00 | 88 | GLOBAL |  | sceVif1PkCnt | sceVif1PkCnt |
| 0x003bfe58 | 152 | GLOBAL |  | sceVif1PkRef | sceVif1PkRef |
| 0x003bfef0 | 112 | GLOBAL |  | sceVif1PkCall | sceVif1PkCall |
| 0x003bff60 | 88 | GLOBAL |  | sceVif1PkRet | sceVif1PkRet |
| 0x003bffb8 | 88 | GLOBAL |  | sceVif1PkEnd | sceVif1PkEnd |
| 0x003c0010 | 144 | GLOBAL |  | sceVif1PkOpenUpkCode | sceVif1PkOpenUpkCode |
| 0x003c00a0 | 80 | GLOBAL |  | sceVif1PkCloseUpkCode | sceVif1PkCloseUpkCode |
| 0x003c00f0 | 20 | GLOBAL |  | sceVif1PkReserve | sceVif1PkReserve |
| 0x003c0108 | 124 | GLOBAL |  | sceVif1PkAlign | sceVif1PkAlign |
| 0x003c0188 | 20 | GLOBAL |  | sceVif1PkAddCode | sceVif1PkAddCode |
| 0x003c01a0 | 20 | GLOBAL |  | sceVif1PkAddData | sceVif1PkAddData |
| 0x003c01b8 | 20 | GLOBAL |  | sceVif1PkAddUpkData32 | sceVif1PkAddUpkData32 |
| 0x003c01d0 | 48 | GLOBAL |  | sceVif1PkAddGsAD | sceVif1PkAddGsAD |
| 0x003c0200 | 116 | GLOBAL |  | sceVif1PkAddUpkData128N | sceVif1PkAddUpkData128N |
| 0x003c0280 | 592 | LOCAL |  | _motionComp0 | _motionComp0 |
| 0x003c04d0 | 1796 | GLOBAL |  | _getAllRefs | _getAllRefs |
| 0x003c0bd8 | 1052 | GLOBAL |  | _getRef0 | _getRef0 |
| 0x003c0ff8 | 540 | LOCAL |  | _doMC | _doMC |
| 0x003c1218 | 116 | GLOBAL |  | _rix_000 | _rix_000 |
| 0x003c1290 | 148 | GLOBAL |  | _ri0_000 | _ri0_000 |
| 0x003c1328 | 180 | GLOBAL |  | _rix_001 | _rix_001 |
| 0x003c13e0 | 204 | GLOBAL |  | _ri0_001 | _ri0_001 |
| 0x003c14b0 | 172 | GLOBAL |  | _rix_010 | _rix_010 |
| 0x003c1560 | 184 | GLOBAL |  | _ri0_010 | _ri0_010 |
| 0x003c1618 | 248 | GLOBAL |  | _rix_011 | _rix_011 |
| 0x003c1710 | 252 | GLOBAL |  | _ri0_011 | _ri0_011 |
| 0x003c1810 | 156 | GLOBAL |  | _rix_100 | _rix_100 |
| 0x003c18b0 | 172 | GLOBAL |  | _ri0_100 | _ri0_100 |
| 0x003c1960 | 220 | GLOBAL |  | _rix_101 | _rix_101 |
| 0x003c1a40 | 228 | GLOBAL |  | _ri0_101 | _ri0_101 |
| 0x003c1b28 | 212 | GLOBAL |  | _rix_110 | _rix_110 |
| 0x003c1c00 | 208 | GLOBAL |  | _ri0_110 | _ri0_110 |
| 0x003c1cd0 | 288 | GLOBAL |  | _rix_111 | _rix_111 |
| 0x003c1df0 | 276 | GLOBAL |  | _ri0_111 | _ri0_111 |
| 0x003c1f08 | 92 | GLOBAL |  | _copyAddRefImage | _copyAddRefImage |
| 0x003c1f68 | 96 | GLOBAL |  | _copyRefImage | _copyRefImage |
| 0x003c1fc8 | 40 | GLOBAL |  | _ipuSetMPEG1 | _ipuSetMPEG1 |
| 0x003c1ff0 | 668 | GLOBAL |  | _waitBdecOut | _waitBdecOut |
| 0x003c2290 | 28 | GLOBAL |  | _dmVector | _dmVector |
| 0x003c22b0 | 388 | GLOBAL |  | _dualPrimeVector | _dualPrimeVector |
| 0x003c2438 | 272 | GLOBAL |  | _mbAddressIncrement | _mbAddressIncrement |
| 0x003c2548 | 288 | GLOBAL |  | _pictureData0 | _pictureData0 |
| 0x003c2668 | 300 | LOCAL |  | _sliceA0 | _sliceA0 |
| 0x003c2798 | 504 | LOCAL |  | _slice0 | _slice0 |
| 0x003c2990 | 188 | LOCAL |  | _skipMB0 | _skipMB0 |
| 0x003c2a50 | 1192 | LOCAL |  | _decMB0 | _decMB0 |
| 0x003c2ef8 | 136 | LOCAL |  | _decode_motion_vector | _decode_motion_vector |
| 0x003c2f80 | 412 | GLOBAL |  | _motionVectors | _motionVectors |
| 0x003c3120 | 332 | GLOBAL |  | _motionVector | _motionVector |
| 0x003c3270 | 44 | GLOBAL |  | _sendIpuCommand | _sendIpuCommand |
| 0x003c32a0 | 164 | GLOBAL |  | _waitIpuIdle | _waitIpuIdle |
| 0x003c3348 | 176 | GLOBAL |  | _waitIpuIdle64 | _waitIpuIdle64 |
| 0x003c33f8 | 376 | GLOBAL |  | _ipuVdec | _ipuVdec |
| 0x003c3570 | 264 | GLOBAL |  | _peepBit | _peepBit |
| 0x003c3678 | 240 | GLOBAL |  | _flushBuf | _flushBuf |
| 0x003c3768 | 336 | GLOBAL |  | _nextBit | _nextBit |
| 0x003c38b8 | 124 | GLOBAL |  | _nextStartCode | _nextStartCode |
| 0x003c3938 | 100 | GLOBAL |  | _sliceB | _sliceB |
| 0x003c39a0 | 268 | GLOBAL |  | _nextHeader | _nextHeader |
| 0x003c3ab0 | 196 | GLOBAL |  | _pictureHeader | _pictureHeader |
| 0x003c3b78 | 296 | GLOBAL |  | _extensionAndUserData | _extensionAndUserData |
| 0x003c3ca0 | 496 | GLOBAL |  | _pictureCodingExtension | _pictureCodingExtension |
| 0x003c3e90 | 72 | GLOBAL |  | _extrainfo | _extrainfo |
| 0x003c3ed8 | 116 | LOCAL |  | _updateTempTackData | _updateTempTackData |
| 0x003c3f50 | 160 | GLOBAL |  | _groupOfPicturesHeader | _groupOfPicturesHeader |
| 0x003c3ff0 | 196 | GLOBAL |  | _quantMatrixExtension | _quantMatrixExtension |
| 0x003c40b8 | 244 | GLOBAL |  | _pictureDisplayExtension | _pictureDisplayExtension |
| 0x003c41b0 | 144 | GLOBAL |  | _copyrightExtension | _copyrightExtension |
| 0x003c4240 | 204 | GLOBAL |  | _decPicture | _decPicture |
| 0x003c4310 | 144 | GLOBAL |  | _outputFrame | _outputFrame |
| 0x003c43a0 | 692 | GLOBAL |  | _updateRefImage | _updateRefImage |
| 0x003c4658 | 160 | LOCAL |  | _isOutSizeOK | _isOutSizeOK |
| 0x003c46f8 | 656 | GLOBAL |  | _cpr8 | _cpr8 |
| 0x003c4988 | 40 | LOCAL |  | _markOutput | _markOutput |
| 0x003c49b0 | 396 | LOCAL |  | _getPtsDtsFlags | _getPtsDtsFlags |
| 0x003c4b40 | 272 | GLOBAL |  | _dispRefImage | _dispRefImage |
| 0x003c4c50 | 440 | GLOBAL |  | _dispRefImageField | _dispRefImageField |
| 0x003c4e08 | 364 | LOCAL |  | dmaRefImage | dmaRefImage |
| 0x003c4f78 | 132 | LOCAL |  | receiveDataFromIPU | receiveDataFromIPU |
| 0x003c5000 | 296 | LOCAL |  | _doCSC | _doCSC |
| 0x003c5128 | 328 | LOCAL |  | _ch3dmaCSC | _ch3dmaCSC |
| 0x003c5270 | 448 | LOCAL |  | _doCSC2 | _doCSC2 |
| 0x003c5430 | 212 | LOCAL |  | _ch4dma | _ch4dma |
| 0x003c5508 | 648 | GLOBAL |  | _csc_storeRefImage | _csc_storeRefImage |
| 0x003c5790 | 52 | GLOBAL |  | _sysbitInit | _sysbitInit |
| 0x003c57c8 | 28 | GLOBAL |  | _sysbitNext | _sysbitNext |
| 0x003c57e8 | 152 | LOCAL |  | _sysbitFlush | _sysbitFlush |
| 0x003c5880 | 76 | GLOBAL |  | _sysbitGet | _sysbitGet |
| 0x003c58d0 | 68 | GLOBAL |  | _sysbitMarker | _sysbitMarker |
| 0x003c5918 | 84 | GLOBAL |  | _sysbitJump | _sysbitJump |
| 0x003c5970 | 44 | GLOBAL |  | _sysbitPtr | _sysbitPtr |
| 0x003c59a0 | 136 | GLOBAL |  | _type2id | _type2id |
| 0x003c5a28 | 508 | GLOBAL |  | _id2type | _id2type |
| 0x003c5c28 | 760 | GLOBAL |  | sceMpegDemuxPssRing | sceMpegDemuxPssRing |
| 0x003c5f20 | 32 | GLOBAL |  | sceMpegDemuxPss | sceMpegDemuxPss |
| 0x003c5f40 | 244 | GLOBAL |  | sceMpegAddStrCallback | sceMpegAddStrCallback |
| 0x003c6038 | 336 | GLOBAL |  | _pack_header | _pack_header |
| 0x003c6188 | 108 | GLOBAL |  | _system_header | _system_header |
| 0x003c61f8 | 1456 | GLOBAL |  | _PES_packet | _PES_packet |
| 0x003c67a8 | 176 | GLOBAL |  | sceMpegInit | sceMpegInit |
| 0x003c6858 | 524 | GLOBAL |  | sceMpegCreate | sceMpegCreate |
| 0x003c6a68 | 8 | GLOBAL |  | sceMpegDelete | sceMpegDelete |
| 0x003c6a70 | 60 | GLOBAL |  | sceMpegAddBs | sceMpegAddBs |
| 0x003c6ab0 | 72 | GLOBAL |  | sceMpegGetPicture | sceMpegGetPicture |
| 0x003c6af8 | 68 | GLOBAL |  | sceMpegGetPictureRAW8 | sceMpegGetPictureRAW8 |
| 0x003c6b40 | 80 | GLOBAL |  | sceMpegGetPictureRAW8xy | sceMpegGetPictureRAW8xy |
| 0x003c6b90 | 16 | GLOBAL |  | sceMpegGetPictureAbort | sceMpegGetPictureAbort |
| 0x003c6ba0 | 20 | GLOBAL |  | sceMpegSetDecodeMode | sceMpegSetDecodeMode |
| 0x003c6bb8 | 32 | GLOBAL |  | sceMpegGetDecodeMode | sceMpegGetDecodeMode |
| 0x003c6bd8 | 12 | GLOBAL |  | sceMpegIsEnd | sceMpegIsEnd |
| 0x003c6be8 | 16 | GLOBAL |  | sceMpegIsRefBuffEmpty | sceMpegIsRefBuffEmpty |
| 0x003c6bf8 | 88 | GLOBAL |  | sceMpegReset | sceMpegReset |
| 0x003c6c50 | 84 | GLOBAL |  | sceMpegClearRefBuff | sceMpegClearRefBuff |
| 0x003c6ca8 | 36 | GLOBAL |  | sceMpegAddCallback | sceMpegAddCallback |
| 0x003c6cd0 | 80 | GLOBAL |  | _dispatchMpegCallback | _dispatchMpegCallback |
| 0x003c6d20 | 40 | GLOBAL |  | _dispatchMpegCbNodata | _dispatchMpegCbNodata |
| 0x003c6d48 | 24 | GLOBAL |  | sceMpegSetDefaultPtsGap | sceMpegSetDefaultPtsGap |
| 0x003c6d60 | 20 | GLOBAL |  | sceMpegResetDefaultPtsGap | sceMpegResetDefaultPtsGap |
| 0x003c6d78 | 16 | GLOBAL |  | sceMpegSetImageBuff | sceMpegSetImageBuff |
| 0x003c6d88 | 12 | GLOBAL |  | sceMpegDispWidth | sceMpegDispWidth |
| 0x003c6d98 | 12 | GLOBAL |  | sceMpegDispHeight | sceMpegDispHeight |
| 0x003c6da8 | 12 | GLOBAL |  | sceMpegDispCenterOffX | sceMpegDispCenterOffX |
| 0x003c6db8 | 12 | GLOBAL |  | sceMpegDispCenterOffY | sceMpegDispCenterOffY |
| 0x003c6dc8 | 16 | GLOBAL |  | sceSetBrokenLink | sceSetBrokenLink |
| 0x003c6dd8 | 24 | GLOBAL |  | sceSetPtm | sceSetPtm |
| 0x003c6df0 | 20 | GLOBAL |  | _alalcInit | _alalcInit |
| 0x003c6e08 | 12 | GLOBAL |  | _alalcSetDynamic | _alalcSetDynamic |
| 0x003c6e18 | 12 | GLOBAL |  | _alalcFree | _alalcFree |
| 0x003c6e28 | 108 | GLOBAL |  | _alalcAlloc | _alalcAlloc |
| 0x003c6e98 | 24 | GLOBAL |  | _alalcRest | _alalcRest |
| 0x003c6eb0 | 368 | LOCAL |  | _getpic | _getpic |
| 0x003c7020 | 276 | LOCAL |  | _decodeOrSkipFrame | _decodeOrSkipFrame |
| 0x003c7138 | 68 | LOCAL |  | _decodeOrSkip | _decodeOrSkip |
| 0x003c7180 | 348 | LOCAL |  | _decodeOrSkipField | _decodeOrSkipField |
| 0x003c72e0 | 104 | GLOBAL |  | _sceMpegFlush | _sceMpegFlush |
| 0x003c7348 | 12 | GLOBAL |  | _initSeqAgain | _initSeqAgain |
| 0x003c7358 | 124 | GLOBAL |  | _lastFrame | _lastFrame |
| 0x003c73d8 | 96 | GLOBAL |  | _clearOnce | _clearOnce |
| 0x003c7438 | 200 | GLOBAL |  | _clearEach | _clearEach |
| 0x003c7500 | 16 | GLOBAL |  | _ErrMessage | _ErrMessage |
| 0x003c7510 | 52 | GLOBAL |  | _Error1 | _Error1 |
| 0x003c7548 | 84 | GLOBAL |  | _Error | _Error |
| 0x003c75a0 | 256 | GLOBAL |  | _sendDataToIPU | _sendDataToIPU |
| 0x003c76a0 | 32 | LOCAL |  | _RefImageInit | _RefImageInit |
| 0x003c76c0 | 292 | GLOBAL |  | _sequenceHeader | _sequenceHeader |
| 0x003c77e8 | 680 | GLOBAL |  | _initSeq | _initSeq |
| 0x003c7a90 | 224 | LOCAL |  | _initRefImages | _initRefImages |
| 0x003c7b70 | 224 | LOCAL |  | _setDefaultQM | _setDefaultQM |
| 0x003c7c50 | 304 | GLOBAL |  | _sequenceExtension | _sequenceExtension |
| 0x003c7d80 | 140 | GLOBAL |  | _sequenceDisplayExtension | _sequenceDisplayExtension |
| 0x003c7e10 | 12 | GLOBAL |  | _sequenceScalableExtension | _sequenceScalableExtension |
| 0x003c7e20 | 12 | GLOBAL |  | _unknown_extension | _unknown_extension |
| 0x003c7e30 | 12 | GLOBAL |  | _pictureSpatialScalableExtension | _pictureSpatialScalableExtension |
| 0x003c7e40 | 12 | GLOBAL |  | _pictureTemporalScalableExtension | _pictureTemporalScalableExtension |
| 0x003c7e50 | 36 | GLOBAL |  | _defStopDMA | _defStopDMA |
| 0x003c7e78 | 36 | GLOBAL |  | _defRestartDMA | _defRestartDMA |
| 0x003c7ea0 | 116 | LOCAL |  | setD3_CHCR | setD3_CHCR |
| 0x003c7f18 | 116 | LOCAL |  | setD4_CHCR | setD4_CHCR |
| 0x003c7f90 | 228 | GLOBAL |  | sceIpuStopDMA | sceIpuStopDMA |
| 0x003c8078 | 336 | GLOBAL |  | sceIpuRestartDMA | sceIpuRestartDMA |
| 0x003c81c8 | 104 | GLOBAL |  | sceIpuSync | sceIpuSync |
| 0x003c8230 | 100 | LOCAL |  | setD4_CHCR | setD4_CHCR |
| 0x003c8298 | 568 | GLOBAL |  | sceIpuInit | sceIpuInit |
| 0x003c84d0 | 164 | GLOBAL |  | sceSdRemoteInit | sceSdRemoteInit |
| 0x003c8578 | 132 | GLOBAL |  | sceSdTransToIOP | sceSdTransToIOP |
| 0x003c8600 | 16 | GLOBAL |  | sceSdCallBack | sceSdCallBack |
| 0x003c8610 | 704 | GLOBAL |  | sceSdRemote | sceSdRemote |
| 0x003c88d0 | 1060 | GLOBAL |  | __ieee754_acosf | __ieee754_acosf |
| 0x003c8cf8 | 928 | GLOBAL |  | __ieee754_asinf | __ieee754_asinf |
| 0x003c9098 | 480 | GLOBAL |  | __ieee754_atan2f | __ieee754_atan2f |
| 0x003c9278 | 464 | GLOBAL |  | __ieee754_fmodf | __ieee754_fmodf |
| 0x003c9448 | 736 | GLOBAL |  | __ieee754_logf | __ieee754_logf |
| 0x003c9728 | 2016 | GLOBAL |  | __ieee754_powf | __ieee754_powf |
| 0x003c9f08 | 944 | GLOBAL |  | __ieee754_rem_pio2f | __ieee754_rem_pio2f |
| 0x003ca2b8 | 204 | GLOBAL |  | __ieee754_sqrtf | __ieee754_sqrtf |
| 0x003ca388 | 336 | GLOBAL |  | __kernel_cosf | __kernel_cosf |
| 0x003ca4d8 | 2148 | GLOBAL |  | __kernel_rem_pio2f | __kernel_rem_pio2f |
| 0x003cad40 | 260 | GLOBAL |  | __kernel_sinf | __kernel_sinf |
| 0x003cae48 | 644 | GLOBAL |  | __kernel_tanf | __kernel_tanf |
| 0x003cb0d0 | 524 | GLOBAL |  | ceil | ceil |
| 0x003cb2e0 | 532 | GLOBAL |  | floor | floor |
| 0x003cb4f8 | 72 | GLOBAL |  | isinf | isinf |
| 0x003cb540 | 60 | GLOBAL |  | isnan | isnan |
| 0x003cb580 | 652 | GLOBAL |  | atanf | atanf |
| 0x003cb810 | 60 | GLOBAL |  | copysignf | copysignf |
| 0x003cb850 | 212 | GLOBAL |  | cosf | cosf |
| 0x003cb928 | 40 | GLOBAL |  | fabsf | fabsf |
| 0x003cb950 | 216 | GLOBAL |  | floorf | floorf |
| 0x003cba28 | 200 | GLOBAL |  | scalbnf | scalbnf |
| 0x003cbaf0 | 232 | GLOBAL |  | sinf | sinf |
| 0x003cbbd8 | 120 | GLOBAL |  | tanf | tanf |
| 0x003cbc50 | 20 | GLOBAL |  | acosf | acosf |
| 0x003cbc68 | 20 | GLOBAL |  | asinf | asinf |
| 0x003cbc80 | 20 | GLOBAL |  | atan2f | atan2f |
| 0x003cbc98 | 20 | GLOBAL |  | fmodf | fmodf |
| 0x003cbcb0 | 20 | GLOBAL |  | logf | logf |
| 0x003cbcc8 | 20 | GLOBAL |  | powf | powf |
| 0x003cbce0 | 36 | LOCAL |  | CB_DelayTh | CB_DelayTh |
| 0x003cbd08 | 104 | GLOBAL |  | sceCdDelayThread | sceCdDelayThread |
| 0x003cbd70 | 104 | GLOBAL |  | sceCdCallback | sceCdCallback |
| 0x003cbdd8 | 160 | GLOBAL |  | _sceCd_cd_callback | _sceCd_cd_callback |
| 0x003cbe78 | 208 | GLOBAL |  | _Cdvd_cbLoop | _Cdvd_cbLoop |
| 0x003cbf48 | 204 | GLOBAL |  | sceCdInitEeCB | sceCdInitEeCB |
| 0x003cc018 | 184 | GLOBAL |  | _sceCd_cd_read_intr | _sceCd_cd_read_intr |
| 0x003cc0d0 | 184 | LOCAL |  | cmd_sem_init | cmd_sem_init |
| 0x003cc188 | 160 | LOCAL |  | cdvd_exit | cdvd_exit |
| 0x003cc228 | 128 | GLOBAL |  | sceCdPOffCallback | sceCdPOffCallback |
| 0x003cc2a8 | 84 | LOCAL |  | _sceCd_Poff_Intr | _sceCd_Poff_Intr |
| 0x003cc300 | 120 | LOCAL |  | PowerOffCB | PowerOffCB |
| 0x003cc378 | 788 | GLOBAL |  | sceCdLayerSearchFile | sceCdLayerSearchFile |
| 0x003cc690 | 28 | GLOBAL |  | sceCdSearchFile | sceCdSearchFile |
| 0x003cc6b0 | 368 | GLOBAL |  | _sceCd_ncmd_prechk | _sceCd_ncmd_prechk |
| 0x003cc820 | 152 | GLOBAL |  | sceCdNcmdDiskReady | sceCdNcmdDiskReady |
| 0x003cc8b8 | 160 | GLOBAL |  | sceCdSync | sceCdSync |
| 0x003cc958 | 108 | GLOBAL |  | sceCdSyncS | sceCdSyncS |
| 0x003cc9c8 | 368 | GLOBAL |  | _sceCd_scmd_prechk | _sceCd_scmd_prechk |
| 0x003ccb38 | 176 | GLOBAL |  | _sceCdSC | _sceCdSC |
| 0x003ccbe8 | 736 | GLOBAL |  | sceCdInit | sceCdInit |
| 0x003ccec8 | 504 | GLOBAL |  | sceCdDiskReady_old | sceCdDiskReady_old |
| 0x003cd0c0 | 516 | GLOBAL |  | sceCdDiskReady | sceCdDiskReady |
| 0x003cd2c8 | 152 | GLOBAL |  | sceCdGetDiskType | sceCdGetDiskType |
| 0x003cd360 | 184 | GLOBAL |  | sceCdStatus | sceCdStatus |
| 0x003cd418 | 208 | GLOBAL |  | sceCdMmode | sceCdMmode |
| 0x003cd4e8 | 248 | GLOBAL |  | sceCdReadClock | sceCdReadClock |
| 0x003cd5e0 | 236 | GLOBAL |  | sceCdTrayReq | sceCdTrayReq |
| 0x003cd6d0 | 44 | GLOBAL |  | sceCdStInit | sceCdStInit |
| 0x003cd700 | 52 | GLOBAL |  | sceCdStStart | sceCdStStart |
| 0x003cd738 | 44 | GLOBAL |  | sceCdStSeekF | sceCdStSeekF |
| 0x003cd768 | 44 | GLOBAL |  | sceCdStSeek | sceCdStSeek |
| 0x003cd798 | 56 | GLOBAL |  | sceCdStStop | sceCdStStop |
| 0x003cd7d0 | 392 | GLOBAL |  | sceCdStRead | sceCdStRead |
| 0x003cd958 | 80 | GLOBAL |  | sceCdStPause | sceCdStPause |
| 0x003cd9a8 | 84 | GLOBAL |  | sceCdStResume | sceCdStResume |
| 0x003cda00 | 72 | GLOBAL |  | sceCdStStat | sceCdStStat |
| 0x003cda48 | 360 | GLOBAL |  | sceCdStream | sceCdStream |
| 0x003cdbb0 | 276 | LOCAL |  | _send_to_iop | _send_to_iop |
| 0x003cdcc8 | 320 | GLOBAL |  | scePadInit | scePadInit |
| 0x003cde08 | 156 | LOCAL |  | scePadInit2 | scePadInit2 |
| 0x003cdea8 | 128 | GLOBAL |  | scePadEnd | scePadEnd |
| 0x003cdf28 | 484 | GLOBAL |  | scePadPortOpen | scePadPortOpen |
| 0x003ce110 | 184 | GLOBAL |  | scePadPortClose | scePadPortClose |
| 0x003ce1c8 | 96 | LOCAL |  | scePadGetDmaStr | scePadGetDmaStr |
| 0x003ce228 | 80 | GLOBAL |  | scePadGetFrameCount | scePadGetFrameCount |
| 0x003ce278 | 356 | GLOBAL |  | scePadRead | scePadRead |
| 0x003ce3e0 | 120 | GLOBAL |  | scePadGetState | scePadGetState |
| 0x003ce458 | 52 | GLOBAL |  | scePadStateIntToStr | scePadStateIntToStr |
| 0x003ce490 | 100 | LOCAL |  | scePadSetReqState | scePadSetReqState |
| 0x003ce4f8 | 80 | GLOBAL |  | scePadGetReqState | scePadGetReqState |
| 0x003ce548 | 52 | GLOBAL |  | scePadReqIntToStr | scePadReqIntToStr |
| 0x003ce580 | 288 | GLOBAL |  | scePadInfoAct | scePadInfoAct |
| 0x003ce6a0 | 284 | GLOBAL |  | scePadInfoComb | scePadInfoComb |
| 0x003ce7c0 | 312 | GLOBAL |  | scePadInfoMode | scePadInfoMode |
| 0x003ce8f8 | 180 | GLOBAL |  | scePadSetMainMode | scePadSetMainMode |
| 0x003ce9b0 | 192 | GLOBAL |  | scePadSetActDirect | scePadSetActDirect |
| 0x003cea70 | 216 | GLOBAL |  | scePadSetActAlign | scePadSetActAlign |
| 0x003ceb48 | 180 | LOCAL |  | scePadGetButtonMask | scePadGetButtonMask |
| 0x003cec00 | 176 | LOCAL |  | scePadSetButtonInfo | scePadSetButtonInfo |
| 0x003cecb0 | 92 | GLOBAL |  | scePadInfoPressMode | scePadInfoPressMode |
| 0x003ced10 | 84 | GLOBAL |  | scePadEnterPressMode | scePadEnterPressMode |
| 0x003ced68 | 84 | GLOBAL |  | scePadExitPressMode | scePadExitPressMode |
| 0x003cedc0 | 204 | LOCAL |  | scePadSetVrefParam | scePadSetVrefParam |
| 0x003cee90 | 100 | LOCAL |  | scePadGetPortMax | scePadGetPortMax |
| 0x003ceef8 | 104 | GLOBAL |  | scePadGetSlotMax | scePadGetSlotMax |
| 0x003cef60 | 100 | LOCAL |  | scePadGetModVersion | scePadGetModVersion |
| 0x003cefc8 | 104 | GLOBAL |  | scePadSetWarningLevel | scePadSetWarningLevel |
| 0x003cf030 | 460 | GLOBAL |  | sceMcInit | sceMcInit |
| 0x003cf200 | 76 | GLOBAL |  | sceMcEnd | sceMcEnd |
| 0x003cf250 | 48 | GLOBAL |  | _lmcGetClientPtr | _lmcGetClientPtr |
| 0x003cf280 | 184 | GLOBAL |  | sceMcChangeThreadPriority | sceMcChangeThreadPriority |
| 0x003cf338 | 196 | GLOBAL |  | sceMcGetSlotMax | sceMcGetSlotMax |
| 0x003cf400 | 292 | GLOBAL |  | sceMcOpen | sceMcOpen |
| 0x003cf528 | 52 | GLOBAL |  | sceMcMkdir | sceMcMkdir |
| 0x003cf560 | 184 | GLOBAL |  | sceMcClose | sceMcClose |
| 0x003cf618 | 216 | GLOBAL |  | sceMcSeek | sceMcSeek |
| 0x003cf6f0 | 144 | LOCAL |  | mceIntrReadFixAlign | mceIntrReadFixAlign |
| 0x003cf780 | 276 | GLOBAL |  | sceMcRead | sceMcRead |
| 0x003cf898 | 372 | GLOBAL |  | sceMcWrite | sceMcWrite |
| 0x003cfa10 | 8 | LOCAL |  | mcHearAlarm | mcHearAlarm |
| 0x003cfa18 | 56 | LOCAL |  | mcDelayThread | mcDelayThread |
| 0x003cfa50 | 224 | GLOBAL |  | sceMcSync | sceMcSync |
| 0x003cfb30 | 84 | LOCAL |  | mceGetInfoApdx | mceGetInfoApdx |
| 0x003cfb88 | 380 | GLOBAL |  | sceMcGetInfo | sceMcGetInfo |
| 0x003cfd08 | 108 | GLOBAL |  | sceMcUdCheckNewCard | sceMcUdCheckNewCard |
| 0x003cfd78 | 328 | GLOBAL |  | sceMcGetDir | sceMcGetDir |
| 0x003cfec0 | 132 | LOCAL |  | mceStorePwd | mceStorePwd |
| 0x003cff48 | 320 | GLOBAL |  | sceMcChdir | sceMcChdir |
| 0x003d0088 | 204 | GLOBAL |  | sceMcFormat | sceMcFormat |
| 0x003d0158 | 280 | GLOBAL |  | sceMcDelete | sceMcDelete |
| 0x003d0270 | 184 | GLOBAL |  | sceMcFlush | sceMcFlush |
| 0x003d0328 | 456 | GLOBAL |  | sceMcSetFileInfo | sceMcSetFileInfo |
| 0x003d04f0 | 344 | GLOBAL |  | sceMcRename | sceMcRename |
| 0x003d0648 | 204 | GLOBAL |  | sceMcUnformat | sceMcUnformat |
| 0x003d0718 | 264 | GLOBAL |  | sceMcGetEntSpace | sceMcGetEntSpace |
| 0x003d0820 | 24 | GLOBAL |  | abort | abort |
| 0x003d0838 | 20 | GLOBAL |  | abs | abs |
| 0x003d0850 | 24 | GLOBAL |  | atof | atof |
| 0x003d0868 | 40 | GLOBAL |  | atoi | atoi |
| 0x003d0890 | 28 | GLOBAL |  | atol | atol |
| 0x003d08b0 | 180 | GLOBAL |  | _calloc_r | _calloc_r |
| 0x003d0968 | 88 | GLOBAL |  | _close_r | _close_r |
| 0x003d09c0 | 556 | LOCAL |  | quorem | quorem |
| 0x003d0bf0 | 4708 | GLOBAL |  | _dtoa_r | _dtoa_r |
| 0x003d1e58 | 12 | GLOBAL |  | __errno | __errno |
| 0x003d1e68 | 172 | GLOBAL |  | exit | exit |
| 0x003d1f18 | 276 | GLOBAL |  | fflush | fflush |
| 0x003d2030 | 92 | LOCAL |  | std | std |
| 0x003d2090 | 120 | GLOBAL |  | __sfmoreglue | __sfmoreglue |
| 0x003d2108 | 248 | GLOBAL |  | __sfp | __sfp |
| 0x003d2200 | 28 | GLOBAL |  | _cleanup_r | _cleanup_r |
| 0x003d2220 | 28 | GLOBAL |  | _cleanup | _cleanup |
| 0x003d2240 | 140 | GLOBAL |  | __sinit | __sinit |
| 0x003d22d0 | 284 | GLOBAL |  | fread | fread |
| 0x003d23f0 | 756 | GLOBAL |  | _free_r | _free_r |
| 0x003d26e8 | 364 | GLOBAL |  | _malloc_trim_r | _malloc_trim_r |
| 0x003d2858 | 1196 | GLOBAL |  | fseek | fseek |
| 0x003d2d08 | 92 | GLOBAL |  | _fstat_r | _fstat_r |
| 0x003d2d68 | 260 | GLOBAL |  | ftell | ftell |
| 0x003d2e70 | 992 | GLOBAL |  | __sfvwrite | __sfvwrite |
| 0x003d3250 | 148 | GLOBAL |  | _fwalk | _fwalk |
| 0x003d32e8 | 140 | GLOBAL |  | _setlocale_r | _setlocale_r |
| 0x003d3378 | 12 | GLOBAL |  | _localeconv_r | _localeconv_r |
| 0x003d3388 | 36 | GLOBAL |  | setlocale | setlocale |
| 0x003d33b0 | 28 | GLOBAL |  | localeconv | localeconv |
| 0x003d33d0 | 96 | GLOBAL |  | _lseek_r | _lseek_r |
| 0x003d3430 | 340 | GLOBAL |  | __smakebuf | __smakebuf |
| 0x003d3588 | 600 | LOCAL |  | malloc_extend_top | malloc_extend_top |
| 0x003d37e0 | 1832 | GLOBAL |  | _malloc_r | _malloc_r |
| 0x003d3f08 | 52 | GLOBAL |  | _mbtowc_r | _mbtowc_r |
| 0x003d3f40 | 224 | GLOBAL |  | memchr | memchr |
| 0x003d4020 | 148 | GLOBAL |  | memcmp | memcmp |
| 0x003d40b8 | 176 | GLOBAL |  | memcpy | memcpy |
| 0x003d4168 | 260 | GLOBAL |  | memmove | memmove |
| 0x003d4270 | 192 | GLOBAL |  | memset | memset |
| 0x003d4330 | 124 | GLOBAL |  | __malloc_lock | __malloc_lock |
| 0x003d43b0 | 76 | GLOBAL |  | __malloc_unlock | __malloc_unlock |
| 0x003d4400 | 164 | GLOBAL |  | _Balloc | _Balloc |
| 0x003d44a8 | 44 | GLOBAL |  | _Bfree | _Bfree |
| 0x003d44d8 | 264 | GLOBAL |  | _multadd | _multadd |
| 0x003d45e0 | 304 | GLOBAL |  | _s2b | _s2b |
| 0x003d4710 | 128 | GLOBAL |  | _hi0bits | _hi0bits |
| 0x003d4790 | 188 | GLOBAL |  | _lo0bits | _lo0bits |
| 0x003d4850 | 56 | GLOBAL |  | _i2b | _i2b |
| 0x003d4888 | 524 | GLOBAL |  | _multiply | _multiply |
| 0x003d4a98 | 252 | GLOBAL |  | _pow5mult | _pow5mult |
| 0x003d4b98 | 344 | GLOBAL |  | _lshift | _lshift |
| 0x003d4cf0 | 104 | GLOBAL |  | __mcmp | __mcmp |
| 0x003d4d58 | 400 | GLOBAL |  | __mdiff | __mdiff |
| 0x003d4ee8 | 208 | GLOBAL |  | _ulp | _ulp |
| 0x003d4fb8 | 408 | GLOBAL |  | _b2d | _b2d |
| 0x003d5150 | 388 | GLOBAL |  | _d2b | _d2b |
| 0x003d52d8 | 204 | GLOBAL |  | _ratio | _ratio |
| 0x003d53a8 | 112 | GLOBAL |  | _mprec_log10 | _mprec_log10 |
| 0x003d5418 | 92 | GLOBAL |  | _printf_r | _printf_r |
| 0x003d5478 | 112 | GLOBAL |  | printf | printf |
| 0x003d54e8 | 2540 | GLOBAL |  | qsort | qsort |
| 0x003d5ed8 | 24 | GLOBAL |  | srand | srand |
| 0x003d5ef0 | 100 | GLOBAL |  | rand | rand |
| 0x003d5f58 | 96 | GLOBAL |  | _read_r | _read_r |
| 0x003d5fb8 | 1440 | GLOBAL |  | _realloc_r | _realloc_r |
| 0x003d6558 | 52 | LOCAL |  | lflush | lflush |
| 0x003d6590 | 380 | GLOBAL |  | __srefill | __srefill |
| 0x003d6710 | 44 | GLOBAL |  | _remove_r | _remove_r |
| 0x003d6740 | 32 | GLOBAL |  | remove | remove |
| 0x003d6760 | 72 | GLOBAL |  | __srget | __srget |
| 0x003d67a8 | 88 | GLOBAL |  | _sbrk_r | _sbrk_r |
| 0x003d6800 | 112 | GLOBAL |  | _init_signal_r | _init_signal_r |
| 0x003d6870 | 128 | GLOBAL |  | _signal_r | _signal_r |
| 0x003d68f0 | 216 | GLOBAL |  | _raise_r | _raise_r |
| 0x003d69c8 | 152 | GLOBAL |  | __sigtramp_r | __sigtramp_r |
| 0x003d6a60 | 32 | GLOBAL |  | raise | raise |
| 0x003d6a80 | 36 | GLOBAL |  | signal | signal |
| 0x003d6aa8 | 28 | GLOBAL |  | _init_signal | _init_signal |
| 0x003d6ac8 | 32 | GLOBAL |  | __sigtramp | __sigtramp |
| 0x003d6ae8 | 92 | GLOBAL |  | _kill_r | _kill_r |
| 0x003d6b48 | 20 | GLOBAL |  | _getpid_r | _getpid_r |
| 0x003d6b60 | 136 | GLOBAL |  | _sprintf_r | _sprintf_r |
| 0x003d6be8 | 144 | GLOBAL |  | sprintf | sprintf |
| 0x003d6c78 | 8 | LOCAL |  | eofread | eofread |
| 0x003d6c80 | 168 | GLOBAL |  | sscanf | sscanf |
| 0x003d6d28 | 100 | GLOBAL |  | __sread | __sread |
| 0x003d6d90 | 128 | GLOBAL |  | __swrite | __swrite |
| 0x003d6e10 | 100 | GLOBAL |  | __sseek | __sseek |
| 0x003d6e78 | 28 | GLOBAL |  | __sclose | __sclose |
| 0x003d6e98 | 216 | GLOBAL |  | strcasecmp | strcasecmp |
| 0x003d6f70 | 300 | GLOBAL |  | strcat | strcat |
| 0x003d70a0 | 400 | GLOBAL |  | strchr | strchr |
| 0x003d7230 | 324 | GLOBAL |  | strcmp | strcmp |
| 0x003d7378 | 276 | GLOBAL |  | strcpy | strcpy |
| 0x003d7490 | 312 | GLOBAL |  | strlen | strlen |
| 0x003d75c8 | 76 | GLOBAL |  | strlwr | strlwr |
| 0x003d7618 | 216 | GLOBAL |  | strncasecmp | strncasecmp |
| 0x003d76f0 | 428 | GLOBAL |  | strncat | strncat |
| 0x003d78a0 | 440 | GLOBAL |  | strncmp | strncmp |
| 0x003d7a58 | 444 | GLOBAL |  | strncpy | strncpy |
| 0x003d7c18 | 80 | GLOBAL |  | strrchr | strrchr |
| 0x003d7c68 | 132 | GLOBAL |  | strstr | strstr |
| 0x003d7cf0 | 3860 | GLOBAL |  | _strtod_r | _strtod_r |
| 0x003d8c08 | 36 | GLOBAL |  | strtod | strtod |
| 0x003d8c30 | 40 | GLOBAL |  | strtodf | strtodf |
| 0x003d8c58 | 568 | GLOBAL |  | _strtol_r | _strtol_r |
| 0x003d8e90 | 44 | GLOBAL |  | strtol | strtol |
| 0x003d8ec0 | 556 | GLOBAL |  | _strtoul_r | _strtoul_r |
| 0x003d90f0 | 44 | GLOBAL |  | strtoul | strtoul |
| 0x003d9120 | 76 | GLOBAL |  | strupr | strupr |
| 0x003d9170 | 32 | GLOBAL |  | tolower | tolower |
| 0x003d9190 | 32 | GLOBAL |  | toupper | toupper |
| 0x003d91b0 | 232 | GLOBAL |  | __submore | __submore |
| 0x003d9298 | 392 | GLOBAL |  | ungetc | ungetc |
| 0x003d9420 | 88 | GLOBAL |  | _unlink_r | _unlink_r |
| 0x003d9478 | 72 | LOCAL |  | __sprint | __sprint |
| 0x003d94c0 | 180 | LOCAL |  | __sbprintf | __sbprintf |
| 0x003d9578 | 116 | GLOBAL |  | vfprintf | vfprintf |
| 0x003d95f0 | 5744 | GLOBAL |  | _vfprintf_r | _vfprintf_r |
| 0x003dac60 | 448 | LOCAL |  | cvt | cvt |
| 0x003dae20 | 220 | LOCAL |  | exponent | exponent |
| 0x003daf00 | 3212 | GLOBAL |  | __svfscanf | __svfscanf |
| 0x003dbb90 | 256 | GLOBAL |  | __sccl | __sccl |
| 0x003dbc90 | 84 | GLOBAL |  | vsprintf | vsprintf |
| 0x003dbce8 | 84 | GLOBAL |  | vsprintf_r | vsprintf_r |
| 0x003dbd40 | 272 | GLOBAL |  | __swbuf | __swbuf |
| 0x003dbe50 | 96 | GLOBAL |  | _write_r | _write_r |
| 0x003dbeb0 | 268 | GLOBAL |  | __swsetup | __swsetup |
| 0x003dbfc0 | 160 | GLOBAL |  | dtaCreate | dtaCreate |
| 0x003dc060 | 560 | GLOBAL | C_rw_data_inter | {ctor} | __ct__15C_rw_data_interFPCcRb |
| 0x003dc290 | 84 | 13 | std::__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> | {dtor} | __dt__Q23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0>Fv |
| 0x003dc2f0 | 20 | 13 | std::__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>> | {ctor} | __ct__Q23std62__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>>Fv |
| 0x003dc310 | 76 | 13 | C_rw_data | {dtor} | __dt__9C_rw_dataFv |
| 0x003dc360 | 584 | GLOBAL | C_rw_data_inter | UnlockPack | UnlockPack__15C_rw_data_interFl |
| 0x003dc5b0 | 8 | GLOBAL | C_rw_data_inter | GetKey | GetKey__15C_rw_data_interFv |
| 0x003dc5c0 | 516 | GLOBAL | C_rw_data_inter | UnlockPackOld | UnlockPackOld__15C_rw_data_interFl |
| 0x003dc7d0 | 492 | GLOBAL | C_rw_data_inter | {dtor} | __dt__15C_rw_data_interFv |
| 0x003dc9c0 | 68 | GLOBAL | C_rw_data_inter | AddRef | AddRef__15C_rw_data_interFv |
| 0x003dca10 | 120 | GLOBAL | C_rw_data_inter | Release | Release__15C_rw_data_interFv |
| 0x003dca90 | 12 | GLOBAL | C_rw_data_inter | SetDtaFirstForce | SetDtaFirstForce__15C_rw_data_interFb |
| 0x003dcaa0 | 168 | GLOBAL |  | dtaOpen | dtaOpen |
| 0x003dcb50 | 1488 | 13 |  | OpenFromDTA__FPCc | OpenFromDTA__FPCc |
| 0x003dd120 | 460 | 13 |  | OpenFromFileSystem__FPCc | OpenFromFileSystem__FPCc |
| 0x003dd2f0 | 296 | 13 |  | GetFirstIndex__FiUi | GetFirstIndex__FiUi |
| 0x003dd420 | 360 | GLOBAL |  | dtaClose | dtaClose |
| 0x003dd590 | 40 | GLOBAL |  | dtaDelete | dtaDelete |
| 0x003dd5c0 | 4244 | GLOBAL |  | dtaRead | dtaRead |
| 0x003de660 | 936 | GLOBAL |  | dtaSeek | dtaSeek |
| 0x003dea10 | 16 | GLOBAL |  | dtaOpenWrite | dtaOpenWrite |
| 0x003dea20 | 400 | GLOBAL |  | dtaWrite | dtaWrite |
| 0x003debb0 | 264 | GLOBAL |  | dtaGetTime | dtaGetTime |
| 0x003decc0 | 40 | GLOBAL | C_multithrerad_protection | {ctor} | __ct__25C_multithrerad_protectionFv |
| 0x003decf0 | 84 | GLOBAL | C_multithrerad_protection | {dtor} | __dt__25C_multithrerad_protectionFv |
| 0x003ded50 | 876 | GLOBAL |  | WavDecompression__FPUcPUcUiP11S_WAVHEADER | WavDecompression__FPUcPUcUiP11S_WAVHEADER |
| 0x003df0c0 | 416 | 13 |  | Decompression__FPUcPUcUi | Decompression__FPUcPUcUi |
| 0x003df260 | 240 | GLOBAL |  | DeCrypt__FPUcUil | DeCrypt__FPUcUil |
| 0x003df350 | 172 | GLOBAL |  | DeCryptAlligned__FPUcUil | DeCryptAlligned__FPUcUil |
| 0x003df400 | 124 | WEAK | std::__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> | resize | resize__Q23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0>FUiRC10S_FILES_IN |
| 0x003df480 | 1228 | WEAK | std::__vector_imp<12S_FILEHANDLE,Q23std25allocator<12S_FILEHANDLE>,0> | insert | insert__Q23std64__vector_imp<12S_FILEHANDLE,Q23std25allocator<12S_FILEHANDLE>,0>FP12S_FILEHANDLEUiRC12S_FILEHANDLE |
| 0x003df950 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std25allocator<12S_FILEHANDLE>,Ui,1> | swap | swap__Q310Metrowerks7details59compressed_pair_imp<Q23std25allocator<12S_FILEHANDLE>,Ui,1>FRQ310Metrowerks7details59compressed_pair_imp<Q23std25allocator<12S_FILEHANDLE>,Ui,1> |
| 0x003df970 | 160 | 13 | S_FILEHANDLE | operator= | __as__12S_FILEHANDLEFRC12S_FILEHANDLE |
| 0x003dfa10 | 292 | WEAK | std::__vector_imp<8S_HANDLE,Q23std20allocator<8S_HANDLE>,0> | erase | erase__Q23std54__vector_imp<8S_HANDLE,Q23std20allocator<8S_HANDLE>,0>FP8S_HANDLE |
| 0x003dfb40 | 80 | 13 | std::__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> | operator= | __as__Q23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0>FRCQ23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> |
| 0x003dfb90 | 1844 | WEAK | std::__vector_imp<8S_HANDLE,Q23std20allocator<8S_HANDLE>,0> | insert | insert__Q23std54__vector_imp<8S_HANDLE,Q23std20allocator<8S_HANDLE>,0>FP8S_HANDLEUiRC8S_HANDLE |
| 0x003e02d0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std20allocator<8S_HANDLE>,Ui,1> | swap | swap__Q310Metrowerks7details54compressed_pair_imp<Q23std20allocator<8S_HANDLE>,Ui,1>FRQ310Metrowerks7details54compressed_pair_imp<Q23std20allocator<8S_HANDLE>,Ui,1> |
| 0x003e02f0 | 28 | 13 | std::__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> | end | end__Q23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0>CFv |
| 0x003e0310 | 8 | 13 | std::__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> | begin | begin__Q23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0>CFv |
| 0x003e0320 | 20 | 13 | std::__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>> | {ctor} | __ct__Q23std62__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>>FRCQ23std23allocator<10S_FILES_IN> |
| 0x003e0340 | 8 | 13 | std::__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>> | alloc | alloc__Q23std62__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>>CFv |
| 0x003e0350 | 68 | WEAK | std::__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>> | clear | clear__Q23std62__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>>Fv |
| 0x003e03a0 | 104 | WEAK | std::__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>> | {dtor} | __dt__Q23std62__vector_deleter<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>>Fv |
| 0x003e0410 | 316 | 13 | std::__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> | init<PC10S_FILES_IN> | init<PC10S_FILES_IN>__Q23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0>FPC10S_FILES_INPC10S_FILES_INQ23std20forward_iterator_tag |
| 0x003e0550 | 684 | 13 | std::__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> | do_assign<PC10S_FILES_IN> | do_assign<PC10S_FILES_IN>__Q23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0>FPC10S_FILES_INPC10S_FILES_INQ23std20forward_iterator_tag |
| 0x003e0800 | 272 | WEAK | std::__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> | erase | erase__Q23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0>FP10S_FILES_INP10S_FILES_IN |
| 0x003e0910 | 1296 | WEAK | std::__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0> | insert | insert__Q23std60__vector_imp<10S_FILES_IN,Q23std23allocator<10S_FILES_IN>,0>FP10S_FILES_INUiRC10S_FILES_IN |
| 0x003e0e20 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std23allocator<10S_FILES_IN>,Ui,1> | swap | swap__Q310Metrowerks7details57compressed_pair_imp<Q23std23allocator<10S_FILES_IN>,Ui,1>FRQ310Metrowerks7details57compressed_pair_imp<Q23std23allocator<10S_FILES_IN>,Ui,1> |
| 0x003e0e40 | 104 | WEAK | std::__vector_deleter<12S_FILEHANDLE,Q23std25allocator<12S_FILEHANDLE>> | {dtor} | __dt__Q23std66__vector_deleter<12S_FILEHANDLE,Q23std25allocator<12S_FILEHANDLE>>Fv |
| 0x003e0eb0 | 104 | WEAK | std::__vector_deleter<8S_HANDLE,Q23std20allocator<8S_HANDLE>> | {dtor} | __dt__Q23std56__vector_deleter<8S_HANDLE,Q23std20allocator<8S_HANDLE>>Fv |
| 0x003e0f20 | 124 | WEAK | std::__vector_deleter<8S_HANDLE,Q23std20allocator<8S_HANDLE>> | clear | clear__Q23std56__vector_deleter<8S_HANDLE,Q23std20allocator<8S_HANDLE>>Fv |
| 0x003e0fa0 | 76 | WEAK | std::__vector_deleter<12S_FILEHANDLE,Q23std25allocator<12S_FILEHANDLE>> | clear | clear__Q23std66__vector_deleter<12S_FILEHANDLE,Q23std25allocator<12S_FILEHANDLE>>Fv |
| 0x003e0ff0 | 92 | 13 | std::vector<8S_HANDLE,Q23std20allocator<8S_HANDLE>> | {dtor} | __dt__Q23std46vector<8S_HANDLE,Q23std20allocator<8S_HANDLE>>Fv |
| 0x003e1050 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std20allocator<8S_HANDLE>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details54compressed_pair_imp<Q23std20allocator<8S_HANDLE>,Ui,1>FUi |
| 0x003e1060 | 92 | 13 | std::vector<12S_FILEHANDLE,Q23std25allocator<12S_FILEHANDLE>> | {dtor} | __dt__Q23std56vector<12S_FILEHANDLE,Q23std25allocator<12S_FILEHANDLE>>Fv |
| 0x003e10c0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std25allocator<12S_FILEHANDLE>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details59compressed_pair_imp<Q23std25allocator<12S_FILEHANDLE>,Ui,1>FUi |
| 0x003e10d0 | 20 | GLOBAL |  | close | close |
| 0x003e10e8 | 20 | GLOBAL |  | fstat | fstat |
| 0x003e1100 | 20 | GLOBAL |  | getpid | getpid |
| 0x003e1118 | 20 | GLOBAL |  | isatty | isatty |
| 0x003e1130 | 20 | GLOBAL |  | kill | kill |
| 0x003e1148 | 20 | GLOBAL |  | lseek | lseek |
| 0x003e1160 | 20 | GLOBAL |  | read | read |
| 0x003e1178 | 52 | GLOBAL |  | sbrk | sbrk |
| 0x003e11b0 | 20 | GLOBAL |  | unlink | unlink |
| 0x003e11c8 | 20 | GLOBAL |  | write | write |
| 0x003e11e0 | 88 | GLOBAL |  | LSISPR_InitTSSArr__FUi | LSISPR_InitTSSArr__FUi |
| 0x003e1240 | 44 | GLOBAL |  | LockTempSPR__FUi | LockTempSPR__FUi |
| 0x003e1270 | 8 | GLOBAL |  | UnlockTempSPR__Fv | UnlockTempSPR__Fv |
| 0x003e1280 | 212 | GLOBAL |  | LockEntireSPR__Fv | LockEntireSPR__Fv |
| 0x003e1360 | 232 | GLOBAL |  | UnlockEntireSPR__Fv | UnlockEntireSPR__Fv |
| 0x003e1450 | 684 | GLOBAL | LSIDevice | UpdateVU1GlobalData_SPR | UpdateVU1GlobalData_SPR__9LSIDeviceFbUi |
| 0x003e1700 | 2084 | GLOBAL | LSIDevice | SendVerticesToVU1_SPR | SendVerticesToVU1_SPR__9LSIDeviceFR14SENDTOVU1_DESC |
| 0x003e1f30 | 180 | GLOBAL | LSIDevice | SendTrianglesToGS_VU1_SPR | SendTrianglesToGS_VU1_SPR__9LSIDeviceFR14SENDTOVU1_DESC |
| 0x003e1ff0 | 108 | GLOBAL | LSIDevice | SendLinesToGS_VU1_SPR | SendLinesToGS_VU1_SPR__9LSIDeviceFR14SENDTOVU1_DESC |
| 0x003e2060 | 716 | GLOBAL | LSIDevice | DrawIndexedPrimitiveUP | DrawIndexedPrimitiveUP__9LSIDeviceF17_LSIPRIMITIVETYPEUiUiUiPCv10_LSIFORMATPCvUi |
| 0x003e2330 | 512 | GLOBAL | LSIDevice | DrawIndexedPrimitive | DrawIndexedPrimitive__9LSIDeviceF17_LSIPRIMITIVETYPEUiUiUiUi |
| 0x003e2530 | 240 | GLOBAL | LSIDevice | DrawPrimitive | DrawPrimitive__9LSIDeviceF17_LSIPRIMITIVETYPEUiUi |
| 0x003e2620 | 632 | GLOBAL | LSIDevice | DrawPrimitiveUP | DrawPrimitiveUP__9LSIDeviceF17_LSIPRIMITIVETYPEUiPCvUi |
| 0x003e28a0 | 236 | GLOBAL |  | FQLoadBegin__FPCc | FQLoadBegin__FPCc |
| 0x003e2990 | 188 | GLOBAL |  | FQLoadEnd__Fv | FQLoadEnd__Fv |
| 0x003e2a50 | 16 | GLOBAL |  | FQL_Seek__Fiii | FQL_Seek__Fiii |
| 0x003e2a60 | 20 | GLOBAL |  | FQL_WriteToFQL___FPvUi | FQL_WriteToFQL___FPvUi |
| 0x003e2a80 | 132 | GLOBAL |  | FQL_ReadFromFQL___FPvUi | FQL_ReadFromFQL___FPvUi |
| 0x003e2b10 | 300 | GLOBAL |  | FQL_Read__FiPvUi | FQL_Read__FiPvUi |
| 0x003e2c40 | 136 | GLOBAL | LSI | Release | Release__3LSIFv |
| 0x003e2cd0 | 144 | GLOBAL | LSI | GetAdapterIdentifier | GetAdapterIdentifier__3LSIFUiUiP22_LSIADAPTER_IDENTIFIER |
| 0x003e2d60 | 116 | GLOBAL | LSI | GetAdapterModeCount | GetAdapterModeCount__3LSIFUi |
| 0x003e2de0 | 1080 | GLOBAL | LSI | EnumAdapterModes | EnumAdapterModes__3LSIFUiUiP15_LSIDISPLAYMODE |
| 0x003e3220 | 172 | GLOBAL | LSI | GetAdapterDisplayMode | GetAdapterDisplayMode__3LSIFUiP15_LSIDISPLAYMODE |
| 0x003e32d0 | 612 | GLOBAL | LSI | CheckDeviceFormat | CheckDeviceFormat__3LSIFUi11_LSIDEVTYPE10_LSIFORMATUi16_LSIRESOURCETYPE10_LSIFORMAT |
| 0x003e3540 | 112 | GLOBAL | LSI | CheckDeviceMultiSampleType | CheckDeviceMultiSampleType__3LSIFUi11_LSIDEVTYPE10_LSIFORMATi20_LSIMULTISAMPLE_TYPE |
| 0x003e35b0 | 244 | GLOBAL | LSI | CheckDepthStencilMatch | CheckDepthStencilMatch__3LSIFUi11_LSIDEVTYPE10_LSIFORMAT10_LSIFORMAT10_LSIFORMAT |
| 0x003e36b0 | 472 | GLOBAL | LSI | GetDeviceCaps | GetDeviceCaps__3LSIFUi11_LSIDEVTYPEP9_LSICAPS8 |
| 0x003e3890 | 200 | GLOBAL | LSI | CreateDevice | CreateDevice__3LSIFUi11_LSIDEVTYPEPvUiP23_LSIPRESENT_PARAMETERS_PP10ILSIDevice |
| 0x003e3960 | 184 | GLOBAL |  | LSICreate__FUi | LSICreate__FUi |
| 0x003e3a20 | 52 | GLOBAL |  | LSIGetErrorStringA__Fi | LSIGetErrorStringA__Fi |
| 0x003e3a60 | 96 | GLOBAL |  | GetFreeHandle__Fi | GetFreeHandle__Fi |
| 0x003e3ac0 | 416 | GLOBAL |  | LSI_fopen | LSI_fopen |
| 0x003e3c60 | 104 | GLOBAL |  | LSI_fclose | LSI_fclose |
| 0x003e3cd0 | 108 | GLOBAL |  | LSI_fprintf | LSI_fprintf |
| 0x003e3d40 | 84 | GLOBAL |  | LSI_fread | LSI_fread |
| 0x003e3da0 | 8 | GLOBAL |  | LSI_fwrite | LSI_fwrite |
| 0x003e3db0 | 16 | GLOBAL |  | LSI_ftell | LSI_ftell |
| 0x003e3dc0 | 88 | GLOBAL |  | LSI_fseek | LSI_fseek |
| 0x003e3e20 | 8 | GLOBAL |  | LSI_dbgClear__Fv | LSI_dbgClear__Fv |
| 0x003e3e30 | 40 | GLOBAL |  | LSI_dbgPrintf__FPCce | LSI_dbgPrintf__FPCce |
| 0x003e3e60 | 8 | GLOBAL |  | LSI_dbgExit__FPCcb | LSI_dbgExit__FPCcb |
| 0x003e3e70 | 128 | GLOBAL | LSIInputDevice | Release | Release__14LSIInputDeviceFv |
| 0x003e3ef0 | 124 | GLOBAL | LSIInputDevice | GetCapabilities | GetCapabilities__14LSIInputDeviceFP11LSIIDEVCAPS |
| 0x003e3f70 | 132 | GLOBAL | LSIInputDevice | EnumObjects | EnumObjects__14LSIInputDeviceFPFPC24LSIIDEVICEOBJECTINSTANCEPv_iPvUi |
| 0x003e4000 | 132 | GLOBAL | LSIInputDevice | GetProperty | GetProperty__14LSIInputDeviceFUiP14LSIIPROPHEADER |
| 0x003e4090 | 132 | GLOBAL | LSIInputDevice | SetProperty | SetProperty__14LSIInputDeviceFUiPC14LSIIPROPHEADER |
| 0x003e4120 | 128 | GLOBAL | LSIInputDevice | Acquire | Acquire__14LSIInputDeviceFv |
| 0x003e41a0 | 136 | GLOBAL | LSIInputDevice | Unacquire | Unacquire__14LSIInputDeviceFv |
| 0x003e4230 | 3448 | GLOBAL | LSIInputDevice | GetDeviceState | GetDeviceState__14LSIInputDeviceFUiPv |
| 0x003e4fb0 | 156 | GLOBAL | LSIInputDevice | SetDataFormat | SetDataFormat__14LSIInputDeviceFPC15_LSIIDATAFORMAT |
| 0x003e5050 | 144 | GLOBAL | LSIInputDevice | SetCooperativeLevel | SetCooperativeLevel__14LSIInputDeviceFPvUi |
| 0x003e50e0 | 124 | GLOBAL | LSIInputDevice | RunControlPanel | RunControlPanel__14LSIInputDeviceFPvUi |
| 0x003e5160 | 140 | GLOBAL | LSIInputDevice | CreateEffect | CreateEffect__14LSIInputDeviceFUiPC10LSIIEFFECTPP15ILSIInputEffectPv |
| 0x003e51f0 | 132 | GLOBAL | LSIInputDevice | EnumEffects | EnumEffects__14LSIInputDeviceFPFPC14LSIIEFFECTINFOPv_iPvUi |
| 0x003e5280 | 164 | GLOBAL | LSIInputDevice | GetForceFeedbackState | GetForceFeedbackState__14LSIInputDeviceFPUi |
| 0x003e5330 | 164 | GLOBAL | LSIInputDevice | SendForceFeedbackCommand | SendForceFeedbackCommand__14LSIInputDeviceFUi |
| 0x003e53e0 | 132 | GLOBAL | LSIInputDevice | Poll | Poll__14LSIInputDeviceFv |
| 0x003e5470 | 156 | GLOBAL | LSIInputDevice | EnumEffectsInFile | EnumEffectsInFile__14LSIInputDeviceFPCcPFPC14LSIIFILEEFFECTPv_iPvUi |
| 0x003e5510 | 32 | GLOBAL | LSIInput | Release | Release__8LSIInputFv |
| 0x003e5530 | 124 | GLOBAL | LSIInput | AddRef | AddRef__8LSIInputFv |
| 0x003e55b0 | 556 | GLOBAL | LSIInput | CreateDevice | CreateDevice__8LSIInputFUiPP15ILSIInputDevicePv |
| 0x003e57e0 | 132 | GLOBAL | LSIInput | EnumDevices | EnumDevices__8LSIInputFUiPFPC18LSIIDEVICEINSTANCEPv_iPvUi |
| 0x003e5870 | 148 | GLOBAL | LSIInput | RunControlPanel | RunControlPanel__8LSIInputFPvUi |
| 0x003e5910 | 40 | GLOBAL | LSIInput | CreateDeviceEx | CreateDeviceEx__8LSIInputFUiUiPPvPv |
| 0x003e5940 | 96 | GLOBAL |  | LSIInputCreate__FPvUiUiPPvPv | LSIInputCreate__FPvUiUiPPvPv |
| 0x003e59a0 | 516 | GLOBAL |  | LSI_Setup__FUiUi | LSI_Setup__FUiUi |
| 0x003e5bb0 | 160 | GLOBAL |  | LSI_SystemUpdate__Fv | LSI_SystemUpdate__Fv |
| 0x003e5c50 | 8 | GLOBAL |  | InitializeCriticalSection__FP21_RTL_CRITICAL_SECTION | InitializeCriticalSection__FP21_RTL_CRITICAL_SECTION |
| 0x003e5c60 | 8 | GLOBAL |  | EnterCriticalSection__FP21_RTL_CRITICAL_SECTION | EnterCriticalSection__FP21_RTL_CRITICAL_SECTION |
| 0x003e5c70 | 8 | GLOBAL |  | LeaveCriticalSection__FP21_RTL_CRITICAL_SECTION | LeaveCriticalSection__FP21_RTL_CRITICAL_SECTION |
| 0x003e5c80 | 8 | GLOBAL |  | DeleteCriticalSection__FP21_RTL_CRITICAL_SECTION | DeleteCriticalSection__FP21_RTL_CRITICAL_SECTION |
| 0x003e5c90 | 96 | GLOBAL |  | LSI_MemoryUse_CheckPoint__Fb | LSI_MemoryUse_CheckPoint__Fb |
| 0x003e5cf0 | 80 | GLOBAL |  | LSI_SetDisplayOffset__Fffff | LSI_SetDisplayOffset__Fffff |
| 0x003e5d40 | 136 | GLOBAL |  | sceLseek_FIX__Fiii | sceLseek_FIX__Fiii |
| 0x003e5dd0 | 48 | GLOBAL |  | intHandler1__Fi | intHandler1__Fi |
| 0x003e5e00 | 8 | GLOBAL |  | timeGetTime__Fv | timeGetTime__Fv |
| 0x003e5e10 | 380 | GLOBAL |  | GetDiskType__Fv | GetDiskType__Fv |
| 0x003e5f90 | 636 | GLOBAL |  | LSI_UpdateSystemTime__Fv | LSI_UpdateSystemTime__Fv |
| 0x003e6210 | 8 | GLOBAL |  | LSI_DebugScreenMessage__FPCc | LSI_DebugScreenMessage__FPCc |
| 0x003e6220 | 8 | GLOBAL |  | LSI_DebugBlinkShock__Fiii | LSI_DebugBlinkShock__Fiii |
| 0x003e6230 | 116 | GLOBAL |  | LSI_StartAll__Fv | LSI_StartAll__Fv |
| 0x003e62b0 | 80 | GLOBAL |  | LSI_StopAll__Fv | LSI_StopAll__Fv |
| 0x003e6300 | 848 | GLOBAL |  | LSI_Init1__FiPPcb | LSI_Init1__FiPPcb |
| 0x003e6650 | 376 | GLOBAL |  | LSI_Init2__Fv | LSI_Init2__Fv |
| 0x003e67d0 | 132 | GLOBAL |  | LSI_Close__Fv | LSI_Close__Fv |
| 0x003e6860 | 180 | GLOBAL |  | Handler_VIF1__Fi | Handler_VIF1__Fi |
| 0x003e6920 | 80 | GLOBAL |  | Handler_DMAC_VIF1__Fi | Handler_DMAC_VIF1__Fi |
| 0x003e6970 | 88 | GLOBAL |  | Handler_VU1__Fi | Handler_VU1__Fi |
| 0x003e69d0 | 40 | GLOBAL |  | itoa__FiPci | itoa__FiPci |
| 0x003e6a00 | 8 | GLOBAL |  | GetLastError__Fv | GetLastError__Fv |
| 0x003e6a10 | 84 | GLOBAL |  | Sleep__FUi | Sleep__FUi |
| 0x003e6a70 | 8 | GLOBAL |  | GetModuleHandle__FPCc | GetModuleHandle__FPCc |
| 0x003e6a80 | 20 | GLOBAL |  | SetRect__FP5_RECTiiii | SetRect__FP5_RECTiiii |
| 0x003e6aa0 | 84 | GLOBAL |  | EqualRect__FPC5_RECTPC5_RECT | EqualRect__FPC5_RECTPC5_RECT |
| 0x003e6b00 | 40 | GLOBAL |  | CopyRect__FP5_RECTPC5_RECT | CopyRect__FP5_RECTPC5_RECT |
| 0x003e6b30 | 116 | GLOBAL |  | CharUpper__FPc | CharUpper__FPc |
| 0x003e6bb0 | 184 | GLOBAL |  | _snprintf__FPcUiPCce | _snprintf__FPcUiPCce |
| 0x003e6c70 | 124 | GLOBAL |  | LSI_CdCallbackReq__FUi | LSI_CdCallbackReq__FUi |
| 0x003e6cf0 | 104 | GLOBAL |  | LSI_CdTrayCheck__Fv | LSI_CdTrayCheck__Fv |
| 0x003e6d60 | 188 | GLOBAL |  | LSI_TestOurCD__Fv | LSI_TestOurCD__Fv |
| 0x003e6e20 | 312 | GLOBAL |  | lsiOpen__FPCci | lsiOpen__FPCci |
| 0x003e6f60 | 416 | GLOBAL |  | lsiRead__FiPvi | lsiRead__FiPvi |
| 0x003e7100 | 216 | GLOBAL |  | LSI_CompleteFullName__FPcPc | LSI_CompleteFullName__FPcPc |
| 0x003e71e0 | 56 | GLOBAL |  | LSI_CompleteFullName__FPc | LSI_CompleteFullName__FPc |
| 0x003e7220 | 208 | GLOBAL |  | CreateFile__FPCcUiUiP20_SECURITY_ATTRIBUTESUiUiPv | CreateFile__FPCcUiUiP20_SECURITY_ATTRIBUTESUiUiPv |
| 0x003e72f0 | 140 | GLOBAL |  | ReadFile__FPvPvUiPUiPv | ReadFile__FPvPvUiPUiPv |
| 0x003e7380 | 40 | GLOBAL |  | WriteFile__FPvPCvUiPUiPv | WriteFile__FPvPCvUiPUiPv |
| 0x003e73b0 | 40 | GLOBAL |  | DeleteFile__FPCc | DeleteFile__FPCc |
| 0x003e73e0 | 104 | GLOBAL |  | GetFileTime__FPvP9_FILETIMEP9_FILETIMEP9_FILETIME | GetFileTime__FPvP9_FILETIMEP9_FILETIMEP9_FILETIME |
| 0x003e7450 | 112 | GLOBAL |  | CloseHandle__FPv | CloseHandle__FPv |
| 0x003e74c0 | 40 | GLOBAL |  | CompareFileTime__FPC9_FILETIMEPC9_FILETIME | CompareFileTime__FPC9_FILETIMEPC9_FILETIME |
| 0x003e74f0 | 112 | GLOBAL |  | FindFirstFile__FPCcP14_LSI_FIND_DATA | FindFirstFile__FPCcP14_LSI_FIND_DATA |
| 0x003e7560 | 72 | GLOBAL |  | FindClose__FPv | FindClose__FPv |
| 0x003e75b0 | 80 | GLOBAL |  | FindNextFile__FPvP14_LSI_FIND_DATA | FindNextFile__FPvP14_LSI_FIND_DATA |
| 0x003e7600 | 236 | GLOBAL |  | SetFilePointer__FPviPiUi | SetFilePointer__FPviPiUi |
| 0x003e76f0 | 8 | GLOBAL |  | LoadCursor__FPvPCc | LoadCursor__FPvPCc |
| 0x003e7700 | 8 | GLOBAL |  | GetStockObject__Fi | GetStockObject__Fi |
| 0x003e7710 | 8 | GLOBAL |  | LoadMenu__FPvPCc | LoadMenu__FPvPCc |
| 0x003e7720 | 168 | GLOBAL |  | SetWindowText__FPvPCc | SetWindowText__FPvPCc |
| 0x003e77d0 | 232 | GLOBAL |  | GetCursorPos__FP8tagPOINT | GetCursorPos__FP8tagPOINT |
| 0x003e78c0 | 8 | GLOBAL |  | ClipCursor__FPC5_RECT | ClipCursor__FPC5_RECT |
| 0x003e78d0 | 40 | GLOBAL |  | GetWindowRect__FPvP5_RECT | GetWindowRect__FPvP5_RECT |
| 0x003e7900 | 56 | GLOBAL |  | GetSystemMetrics__Fi | GetSystemMetrics__Fi |
| 0x003e7940 | 92 | GLOBAL |  | GetLocalTime__FP11_SYSTEMTIME | GetLocalTime__FP11_SYSTEMTIME |
| 0x003e79a0 | 64 | GLOBAL |  | ShowCursor__Fi | ShowCursor__Fi |
| 0x003e79e0 | 56 | GLOBAL |  | CreateWindowEx__FUiPCcPCcUiiiiiPvPvPvPv | CreateWindowEx__FUiPCcPCcUiiiiiPvPvPvPv |
| 0x003e7a20 | 40 | GLOBAL |  | MessageBox__FPvPCcPCcUi | MessageBox__FPvPCcPCcUi |
| 0x003e7a50 | 2276 | GLOBAL | LSIDevice | {ctor} | __ct__9LSIDeviceFv |
| 0x003e8340 | 8 | 13 | LSI_SPOTLIGHT | {ctor} | __ct__13LSI_SPOTLIGHTFv |
| 0x003e8350 | 760 | GLOBAL | LSIDevice | Init | Init__9LSIDeviceFP23_LSIPRESENT_PARAMETERS_ |
| 0x003e8650 | 36 | GLOBAL | LSIDevice | Freeze | Freeze__9LSIDeviceFv |
| 0x003e8680 | 1064 | GLOBAL | LSIDevice | ReInit | ReInit__9LSIDeviceFv |
| 0x003e8ab0 | 480 | GLOBAL | LSIDevice | UploadConstGSRegs | UploadConstGSRegs__9LSIDeviceFv |
| 0x003e8c90 | 52 | GLOBAL |  | LSI_FlushRBuff__Fv | LSI_FlushRBuff__Fv |
| 0x003e8cd0 | 1352 | GLOBAL | LSIDevice | RBuffProcessOneDmaCommand | RBuffProcessOneDmaCommand__9LSIDeviceFb |
| 0x003e9220 | 88 | GLOBAL | LSIDevice | AllocRBuff | AllocRBuff__9LSIDeviceFv |
| 0x003e9280 | 48 | GLOBAL | LSIDevice | FreeRBuff | FreeRBuff__9LSIDeviceFv |
| 0x003e92b0 | 256 | GLOBAL | LSIDevice | FlushRBuff | FlushRBuff__9LSIDeviceFb |
| 0x003e93b0 | 8 | GLOBAL | LSIDevice | IsIRenderReady | IsIRenderReady__9LSIDeviceFv |
| 0x003e93c0 | 116 | GLOBAL | LSIDevice | StartIRender | StartIRender__9LSIDeviceFv |
| 0x003e9440 | 192 | GLOBAL | LSIDevice | FirstDrawPrimitiveCallOrDeviceRelease | FirstDrawPrimitiveCallOrDeviceRelease__9LSIDeviceFb |
| 0x003e9500 | 124 | GLOBAL | LSIDevice | Realocate_RBuff_ReleasedResBuffsList | Realocate_RBuff_ReleasedResBuffsList__9LSIDeviceFUi |
| 0x003e9580 | 92 | GLOBAL | LSIDevice | AddReleasedResBuffToList | AddReleasedResBuffToList__9LSIDeviceFPv |
| 0x003e95e0 | 92 | GLOBAL | LSIDevice | FreeUsedTexBuffsAndVifBuffs | FreeUsedTexBuffsAndVifBuffs__9LSIDeviceFv |
| 0x003e9640 | 368 | GLOBAL | LSIDevice | FromMemToRBuff_ref | FromMemToRBuff_ref__9LSIDeviceFUiPvUi |
| 0x003e97b0 | 604 | GLOBAL | LSIDevice | FromSPRToRBuff | FromSPRToRBuff__9LSIDeviceFUiPvUiUi |
| 0x003e9a10 | 204 | GLOBAL | LSIDevice | AddCommandToRBuff | AddCommandToRBuff__9LSIDeviceFUl |
| 0x003e9ae0 | 212 | GLOBAL | LSIDevice | UpdateRefBuffOfLights | UpdateRefBuffOfLights__9LSIDeviceFv |
| 0x003e9bc0 | 524 | GLOBAL | LSIDevice | VU1_UpdateGlobalData | VU1_UpdateGlobalData__9LSIDeviceFv |
| 0x003e9dd0 | 188 | GLOBAL | LSIDevice | Release | Release__9LSIDeviceFv |
| 0x003e9e90 | 108 | GLOBAL | LSIDevice | TestCooperativeLevel | TestCooperativeLevel__9LSIDeviceFv |
| 0x003e9f00 | 208 | GLOBAL | LSIDevice | ResourceManagerDiscardBytes | ResourceManagerDiscardBytes__9LSIDeviceFUi |
| 0x003e9fd0 | 92 | GLOBAL | LSIDevice | Reset | Reset__9LSIDeviceFP23_LSIPRESENT_PARAMETERS_ |
| 0x003ea030 | 512 | GLOBAL | LSIDevice | ClearVRAM | ClearVRAM__9LSIDeviceFUcUcUcUc |
| 0x003ea230 | 1108 | GLOBAL | LSIDevice | ClearFrameBuffer | ClearFrameBuffer__9LSIDeviceFUcUcUcUcUi |
| 0x003ea690 | 384 | GLOBAL | LSIDevice | Send_CRTCMAfterRenderPacket | Send_CRTCMAfterRenderPacket__9LSIDeviceFv |
| 0x003ea810 | 244 | GLOBAL | LSIDevice | Present | Present__9LSIDeviceFPC5_RECTPC5_RECTPvPC8_RGNDATA |
| 0x003ea910 | 140 | GLOBAL |  | Present____9LSIDeviceFv | Present____9LSIDeviceFv |
| 0x003ea9a0 | 92 | GLOBAL | LSIDevice | SetGammaRamp | SetGammaRamp__9LSIDeviceFUiPC13_LSIGAMMARAMP |
| 0x003eaa00 | 92 | GLOBAL | LSIDevice | GetGammaRamp | GetGammaRamp__9LSIDeviceFP13_LSIGAMMARAMP |
| 0x003eaa60 | 296 | GLOBAL | LSIDevice | CreateTexture | CreateTexture__9LSIDeviceFUiUiUiUi10_LSIFORMAT8_LSIPOOLPP11ILSITexture |
| 0x003eab90 | 492 | GLOBAL | LSIDevice | CreateVertexBuffer | CreateVertexBuffer__9LSIDeviceFUiUiUi8_LSIPOOLPP16ILSIVertexBuffer |
| 0x003ead80 | 260 | GLOBAL | LSIDevice | CreateIndexBuffer | CreateIndexBuffer__9LSIDeviceFUiUi10_LSIFORMAT8_LSIPOOLPP15ILSIIndexBuffer |
| 0x003eae90 | 108 | GLOBAL | LSIDevice | CreateImageSurface | CreateImageSurface__9LSIDeviceFUiUi10_LSIFORMATPP11ILSISurface |
| 0x003eaf00 | 100 | GLOBAL | LSIDevice | CopyRects | CopyRects__9LSIDeviceFP11ILSISurfacePC5_RECTUiP11ILSISurfacePC8tagPOINT |
| 0x003eaf70 | 100 | GLOBAL | LSIDevice | GetFrontBuffer | GetFrontBuffer__9LSIDeviceFP11ILSISurface |
| 0x003eafe0 | 108 | GLOBAL | LSIDevice | GetRenderTarget | GetRenderTarget__9LSIDeviceFPP11ILSISurface |
| 0x003eb050 | 160 | GLOBAL | LSIDevice | BeginScene | BeginScene__9LSIDeviceFv |
| 0x003eb0f0 | 108 | GLOBAL | LSIDevice | EndScene | EndScene__9LSIDeviceFv |
| 0x003eb160 | 228 | GLOBAL | LSIDevice | Clear | Clear__9LSIDeviceFUiPC8_LSIRECTUiUifUi |
| 0x003eb250 | 180 | GLOBAL |  | Clear____9LSIDeviceFUiUi | Clear____9LSIDeviceFUiUi |
| 0x003eb310 | 1304 | GLOBAL | LSIDevice | SetTransform | SetTransform__9LSIDeviceF22_LSITRANSFORMSTATETYPEPC10_LSIMATRIX |
| 0x003eb830 | 548 | GLOBAL | LSIDevice | SetViewport | SetViewport__9LSIDeviceFPC12_LSIVIEWPORT |
| 0x003eba60 | 1032 | GLOBAL | LSIDevice | SetMaterial | SetMaterial__9LSIDeviceFPC13_LSIMATERIAL8 |
| 0x003ebe70 | 304 | GLOBAL | LSIDevice | SetLight | SetLight__9LSIDeviceFUiPC10_D3DLIGHT8 |
| 0x003ebfa0 | 656 | GLOBAL |  | SetLight____9LSIDeviceFUiPC10_D3DLIGHT8 | SetLight____9LSIDeviceFUiPC10_D3DLIGHT8 |
| 0x003ec230 | 256 | GLOBAL | LSIDevice | LightEnable | LightEnable__9LSIDeviceFUii |
| 0x003ec330 | 272 | GLOBAL |  | LightEnable____9LSIDeviceFUii | LightEnable____9LSIDeviceFUii |
| 0x003ec440 | 4260 | GLOBAL | LSIDevice | SetRenderState | SetRenderState__9LSIDeviceF19_LSIRENDERSTATETYPEUi |
| 0x003ed4f0 | 100 | GLOBAL | LSIDevice | GetRenderState | GetRenderState__9LSIDeviceF19_LSIRENDERSTATETYPEPUi |
| 0x003ed560 | 288 | GLOBAL | LSIDevice | AddToStateBlock | AddToStateBlock__9LSIDeviceFUiPUi |
| 0x003ed680 | 160 | GLOBAL | LSIDevice | BeginStateBlock | BeginStateBlock__9LSIDeviceFv |
| 0x003ed720 | 188 | GLOBAL | LSIDevice | EndStateBlock | EndStateBlock__9LSIDeviceFPUi |
| 0x003ed7e0 | 316 | GLOBAL | LSIDevice | ApplyStateBlock | ApplyStateBlock__9LSIDeviceFUi |
| 0x003ed920 | 144 | GLOBAL | LSIDevice | DeleteStateBlock | DeleteStateBlock__9LSIDeviceFUi |
| 0x003ed9b0 | 876 | GLOBAL | LSIDevice | SetTexture | SetTexture__9LSIDeviceFUiP11ILSITexture |
| 0x003edd20 | 428 | GLOBAL | LSIDevice | UpdateRenderStates | UpdateRenderStates__9LSIDeviceFv |
| 0x003eded0 | 8 | GLOBAL | LSIDevice | UpdateLSIStates | UpdateLSIStates__9LSIDeviceFv |
| 0x003edee0 | 708 | GLOBAL | LSIDevice | ProcessTextureStageStates | ProcessTextureStageStates__9LSIDeviceFv |
| 0x003ee1b0 | 1700 | GLOBAL | LSIDevice | ProvideTextures | ProvideTextures__9LSIDeviceFv |
| 0x003ee860 | 724 | GLOBAL | LSIDevice | SetTextureStageState | SetTextureStageState__9LSIDeviceFUi25_LSITEXTURESTAGESTATETYPEUi |
| 0x003eeb40 | 108 | GLOBAL | LSIDevice | ProcessVertices | ProcessVertices__9LSIDeviceFUiUiUiP16ILSIVertexBufferUi |
| 0x003eebb0 | 120 | GLOBAL | LSIDevice | CreateVertexShader | CreateVertexShader__9LSIDeviceFPCUiPCUiPUiUi |
| 0x003eec30 | 1088 | GLOBAL | LSIDevice | SetVertexShader | SetVertexShader__9LSIDeviceFUi |
| 0x003ef070 | 120 | GLOBAL | LSIDevice | DeleteVertexShader | DeleteVertexShader__9LSIDeviceFUi |
| 0x003ef0f0 | 432 | GLOBAL | LSIDevice | SetStreamSource | SetStreamSource__9LSIDeviceFUiP16ILSIVertexBufferUi |
| 0x003ef2a0 | 160 | GLOBAL | LSIDevice | SetIndices | SetIndices__9LSIDeviceFP15ILSIIndexBufferUi |
| 0x003ef340 | 56 | GLOBAL | LSIDevice | VU1uCodeUpload | VU1uCodeUpload__9LSIDeviceFRUi |
| 0x003ef380 | 92 | GLOBAL | LSIDevice | DmaSendToVif | DmaSendToVif__9LSIDeviceFUi |
| 0x003ef3e0 | 76 | GLOBAL | LSIIndexBuffer | InitVifBuffStructs | InitVifBuffStructs__14LSIIndexBufferFv |
| 0x003ef430 | 308 | GLOBAL | LSIIndexBuffer | Init | Init__14LSIIndexBufferFUiUi10_LSIFORMAT |
| 0x003ef570 | 152 | GLOBAL | LSIIndexBuffer | ReleaseVifBuffs | ReleaseVifBuffs__14LSIIndexBufferFv |
| 0x003ef610 | 204 | GLOBAL | LSIIndexBuffer | Release | Release__14LSIIndexBufferFv |
| 0x003ef6e0 | 180 | GLOBAL | LSIIndexBuffer | Lock | Lock__14LSIIndexBufferFUiUiPPUcUi |
| 0x003ef7a0 | 152 | GLOBAL | LSIIndexBuffer | Unlock | Unlock__14LSIIndexBufferFv |
| 0x003ef840 | 2732 | GLOBAL | LSISurface | Init | Init__10LSISurfaceFUiUiUi10_LSIFORMAT8_LSIPOOL |
| 0x003f02f0 | 108 | GLOBAL | LSISurface | AddRef | AddRef__10LSISurfaceFv |
| 0x003f0360 | 192 | GLOBAL | LSISurface | Release | Release__10LSISurfaceFv |
| 0x003f0420 | 188 | GLOBAL | LSISurface | GetDesc | GetDesc__10LSISurfaceFP16_LSISURFACE_DESC |
| 0x003f04e0 | 668 | GLOBAL | LSISurface | LockRect | LockRect__10LSISurfaceFP15_LSILOCKED_RECTPC5_RECTUi |
| 0x003f0780 | 400 | GLOBAL | LSISurface | UnlockRect | UnlockRect__10LSISurfaceFv |
| 0x003f0910 | 324 | GLOBAL | LSITexture | Init | Init__10LSITextureFUiUiUiUi10_LSIFORMAT8_LSIPOOL |
| 0x003f0a60 | 144 | GLOBAL | LSITexture | Release | Release__10LSITextureFv |
| 0x003f0af0 | 92 | GLOBAL | LSITexture | PreLoad | PreLoad__10LSITextureFv |
| 0x003f0b50 | 132 | GLOBAL | LSITexture | GetLevelCount | GetLevelCount__10LSITextureFv |
| 0x003f0be0 | 208 | GLOBAL | LSITexture | GetLevelDesc | GetLevelDesc__10LSITextureFUiP16_LSISURFACE_DESC |
| 0x003f0cb0 | 204 | GLOBAL | LSITexture | GetSurfaceLevel | GetSurfaceLevel__10LSITextureFUiPP11ILSISurface |
| 0x003f0d80 | 116 | GLOBAL | LSITexture | AddDirtyRect | AddDirtyRect__10LSITextureFPC5_RECT |
| 0x003f0e00 | 72 | GLOBAL | LSIVertexBuffer | Realloc | Realloc__15LSIVertexBufferFUi |
| 0x003f0e50 | 8 | GLOBAL | LSIVertexBuffer | GetData | GetData__15LSIVertexBufferFv |
| 0x003f0e60 | 196 | GLOBAL | LSIVertexBuffer | Init | Init__15LSIVertexBufferFUiUiUi |
| 0x003f0f30 | 96 | GLOBAL | LSIVertexBuffer | AddRef | AddRef__15LSIVertexBufferFv |
| 0x003f0f90 | 136 | GLOBAL | LSIVertexBuffer | Release | Release__15LSIVertexBufferFv |
| 0x003f1020 | 224 | GLOBAL | LSIVertexBuffer | Lock | Lock__15LSIVertexBufferFUiUiPPUcUi |
| 0x003f1100 | 108 | GLOBAL | LSIVertexBuffer | Unlock | Unlock__15LSIVertexBufferFv |
| 0x003f1170 | 392 | GLOBAL |  | FMVDisplayInit__F15EFMVDisplayModeiPUiPUi | FMVDisplayInit__F15EFMVDisplayModeiPUiPUi |
| 0x003f1300 | 248 | GLOBAL |  | FMVDisplayCreate__FUiUi | FMVDisplayCreate__FUiUi |
| 0x003f1400 | 112 | GLOBAL |  | FMVDisplayAddFrame__FP12_ShellDmaTag | FMVDisplayAddFrame__FP12_ShellDmaTag |
| 0x003f1470 | 76 | GLOBAL |  | FMVDisplayStart__Fv | FMVDisplayStart__Fv |
| 0x003f14c0 | 184 | GLOBAL |  | FMVDisplayFlush__Fv | FMVDisplayFlush__Fv |
| 0x003f1580 | 8 | GLOBAL |  | FMVDisplayDestroy__Fv | FMVDisplayDestroy__Fv |
| 0x003f1590 | 292 | LOCAL |  | vSyncCallbackInterlace__Fi | vSyncCallbackInterlace__Fi |
| 0x003f16c0 | 288 | LOCAL |  | vSyncCallbackPalFromNtsc__Fi | vSyncCallbackPalFromNtsc__Fi |
| 0x003f17e0 | 156 | LOCAL |  | vSyncCallbackProgressive__Fi | vSyncCallbackProgressive__Fi |
| 0x003f1880 | 384 | LOCAL |  | spriteCreate__FUiUiUiUiUiUiUii | spriteCreate__FUiUiUiUiUiUiUii |
| 0x003f1a00 | 48 | GLOBAL |  | FMVAudioInit__Fv | FMVAudioInit__Fv |
| 0x003f1a30 | 188 | GLOBAL |  | FMVAudioCreate__F19EShellMpegAudioTypeP18ShellMpegADSHeader | FMVAudioCreate__F19EShellMpegAudioTypeP18ShellMpegADSHeader |
| 0x003f1af0 | 120 | GLOBAL |  | FMVAudioSend__FPUcUiUiUii | FMVAudioSend__FPUcUiUiUii |
| 0x003f1b70 | 108 | GLOBAL |  | FMVAudioIsStartable__Fv | FMVAudioIsStartable__Fv |
| 0x003f1be0 | 564 | GLOBAL |  | FMVAudioStart__Fv | FMVAudioStart__Fv |
| 0x003f1e20 | 288 | GLOBAL |  | FMVAudioDestroy__Fv | FMVAudioDestroy__Fv |
| 0x003f1f40 | 112 | LOCAL |  | changeMasterVolume__FUi | changeMasterVolume__FUi |
| 0x003f1fb0 | 72 | LOCAL |  | changePcmVolume__FUi | changePcmVolume__FUi |
| 0x003f2000 | 216 | LOCAL |  | changeAdpcmVolume__FUi | changeAdpcmVolume__FUi |
| 0x003f20e0 | 372 | LOCAL |  | sendToIOP2area__FiiiiPUciPUci | sendToIOP2area__FiiiiPUciPUci |
| 0x003f2260 | 116 | LOCAL |  | sendToIOP__FiPUcii | sendToIOP__FiPUcii |
| 0x003f22e0 | 116 | LOCAL |  | sendToSPU__FPUcii | sendToSPU__FPUcii |
| 0x003f2360 | 176 | LOCAL |  | adpcmDeinterleave__FP1P1UiUiUi | adpcmDeinterleave__FP1P1UiUiUi |
| 0x003f2410 | 348 | LOCAL |  | sendPCM__FPUcUiUiUi | sendPCM__FPUcUiUiUi |
| 0x003f2570 | 544 | LOCAL |  | sendADPCM__FPUcUiUiUi | sendADPCM__FPUcUiUiUi |
| 0x003f2790 | 332 | LOCAL |  | sendSPDIF__FPUcUiUiUi | sendSPDIF__FPUcUiUiUi |
| 0x003f28e0 | 1496 | GLOBAL | LSIDevice | AnalyzeVBForVifBuff | AnalyzeVBForVifBuff__9LSIDeviceFUi |
| 0x003f2ec0 | 368 | GLOBAL | LSIDevice | CreateVifBuff_ls3d | CreateVifBuff_ls3d__9LSIDeviceFPUsUiPUiUi |
| 0x003f3030 | 836 | GLOBAL |  | LSI_AddVU1UpdateTag__FR13sceVif1Packet | LSI_AddVU1UpdateTag__FR13sceVif1Packet |
| 0x003f3380 | 368 | GLOBAL | LSIDevice | DrawVifBuff | DrawVifBuff__9LSIDeviceFUi |
| 0x003f34f0 | 8 | GLOBAL |  | DeleteVifBuff___FUi | DeleteVifBuff___FUi |
| 0x003f3500 | 216 | GLOBAL | LSIDevice | DeleteVifBuff_ls3d | DeleteVifBuff_ls3d__9LSIDeviceFPUi |
| 0x003f35e0 | 8112 | GLOBAL |  | CreateVifBuffPacket__FPUsUiPCvUiUiUiUiUi | CreateVifBuffPacket__FPUsUiPCvUiUiUiUiUi |
| 0x003f5590 | 556 | GLOBAL | LSIDevice | CreateSngmVifBuff_ls3d | CreateSngmVifBuff_ls3d__9LSIDeviceFPvUiPUi |
| 0x003f57c0 | 156 | GLOBAL | LSIDevice | SendSngmVifBuff | SendSngmVifBuff__9LSIDeviceFUi |
| 0x003f5860 | 208 | GLOBAL | LSIDevice | DeleteSngmVifBuff_ls3d | DeleteSngmVifBuff_ls3d__9LSIDeviceFPUi |
| 0x003f5930 | 84 | GLOBAL | LSIDevice | SngmBlendDataBegin | SngmBlendDataBegin__9LSIDeviceFPvUiUiPv |
| 0x003f5990 | 36 | GLOBAL | LSIDevice | SngmBlendDataEnd | SngmBlendDataEnd__9LSIDeviceFv |
| 0x003f59c0 | 128 | GLOBAL |  | DeleteAllLS3DVifBuffs__Fv | DeleteAllLS3DVifBuffs__Fv |
| 0x003f5a40 | 92 | 13 | C_list<Ui> | {dtor} | __dt__10C_list<Ui>Fv |
| 0x003f5aa0 | 944 | GLOBAL |  | VU1GSR_Init__FP9LSIDevice | VU1GSR_Init__FP9LSIDevice |
| 0x003f5e50 | 1276 | GLOBAL |  | VU1GSR_InitDataInVU1__FP9LSIDevice | VU1GSR_InitDataInVU1__FP9LSIDevice |
| 0x003f6350 | 64 | GLOBAL |  | VU1GSR_Set_FRAMEMASK_1__FUi | VU1GSR_Set_FRAMEMASK_1__FUi |
| 0x003f6390 | 44 | GLOBAL |  | VU1GSR_Set_FRAME_1__Fl | VU1GSR_Set_FRAME_1__Fl |
| 0x003f63c0 | 44 | GLOBAL |  | VU1GSR_Set_FogColor__FUi | VU1GSR_Set_FogColor__FUi |
| 0x003f63f0 | 360 | GLOBAL |  | VU1GSR_UpdateVU1Mem__FP9LSIDevice | VU1GSR_UpdateVU1Mem__FP9LSIDevice |
| 0x003f6560 | 64 | GLOBAL |  | InitSPR__Fv | InitSPR__Fv |
| 0x003f65a0 | 116 | GLOBAL |  | __muldi3 | __muldi3 |
| 0x003f6618 | 1684 | GLOBAL |  | __divdi3 | __divdi3 |
| 0x003f6cb0 | 1680 | GLOBAL |  | __moddi3 | __moddi3 |
| 0x003f7340 | 1404 | GLOBAL |  | __udivdi3 | __udivdi3 |
| 0x003f78c0 | 1384 | GLOBAL |  | __umoddi3 | __umoddi3 |
| 0x003f7e28 | 168 | GLOBAL |  | __floatdidf | __floatdidf |
| 0x003f7ed0 | 280 | GLOBAL |  | __fixunsdfdi | __fixunsdfdi |
| 0x003f7fe8 | 120 | GLOBAL |  | __fixdfdi | __fixdfdi |
| 0x003f8060 | 296 | GLOBAL |  | __fixunssfdi | __fixunssfdi |
| 0x003f8188 | 60 | GLOBAL |  | __fixsfdi | __fixsfdi |
| 0x003f81c8 | 180 | GLOBAL |  | __pack_f | __pack_f |
| 0x003f8280 | 92 | GLOBAL |  | __unpack_f | __unpack_f |
| 0x003f82e0 | 44 | GLOBAL |  | __make_fp | __make_fp |
| 0x003f8310 | 60 | GLOBAL |  | fptodp | fptodp |
| 0x003f8350 | 216 | GLOBAL |  | __pack_d | __pack_d |
| 0x003f8428 | 188 | GLOBAL |  | __unpack_d | __unpack_d |
| 0x003f84e8 | 604 | LOCAL |  | _fpadd_parts | _fpadd_parts |
| 0x003f8748 | 88 | GLOBAL |  | dpadd | dpadd |
| 0x003f87a0 | 100 | GLOBAL |  | dpsub | dpsub |
| 0x003f8808 | 644 | GLOBAL |  | dpmul | dpmul |
| 0x003f8a90 | 320 | GLOBAL |  | dpdiv | dpdiv |
| 0x003f8bd0 | 76 | GLOBAL |  | dpcmp | dpcmp |
| 0x003f8c20 | 184 | GLOBAL |  | litodp | litodp |
| 0x003f8cd8 | 156 | GLOBAL |  | dptoli | dptoli |
| 0x003f8d78 | 44 | GLOBAL |  | __make_dp | __make_dp |
| 0x003f8da8 | 84 | GLOBAL |  | dptofp | dptofp |
| 0x003f8e00 | 160 | GLOBAL |  | dptoul | dptoul |
| 0x003f8ea0 | 260 | GLOBAL |  | FMM_Init__FUiUi | FMM_Init__FUiUi |
| 0x003f8fb0 | 72 | GLOBAL |  | FMM_Close__Fv | FMM_Close__Fv |
| 0x003f9000 | 492 | GLOBAL |  | FMM_Manage_onestep__Fv | FMM_Manage_onestep__Fv |
| 0x003f91f0 | 164 | GLOBAL |  | FMM_RemoveFreeRBlockFromBeginOfChain__Fv | FMM_RemoveFreeRBlockFromBeginOfChain__Fv |
| 0x003f92a0 | 116 | GLOBAL |  | FMM_Manage__Fi | FMM_Manage__Fi |
| 0x003f9320 | 8 | GLOBAL |  | FMM_GetMArea__Fv | FMM_GetMArea__Fv |
| 0x003f9330 | 8 | GLOBAL |  | FMM_GetMAreaSize__Fv | FMM_GetMAreaSize__Fv |
| 0x003f9340 | 124 | GLOBAL |  | FMM_FlushDelete__Fv | FMM_FlushDelete__Fv |
| 0x003f93c0 | 112 | GLOBAL |  | FMM_MAreaConstAlloc__FUi | FMM_MAreaConstAlloc__FUi |
| 0x003f9430 | 68 | GLOBAL |  | FMM_MAreaConstFree__FPv | FMM_MAreaConstFree__FPv |
| 0x003f9480 | 8 | GLOBAL |  | FMM_dbg_AfterRenderFinish__Fv | FMM_dbg_AfterRenderFinish__Fv |
| 0x003f9490 | 24 | GLOBAL |  | FMM_RGetSize__FUi | FMM_RGetSize__FUi |
| 0x003f94b0 | 616 | GLOBAL |  | FMM_RAlloc__FUiUi | FMM_RAlloc__FUiUi |
| 0x003f9720 | 132 | GLOBAL |  | MArea_MarkThisBlockLikeFree__FUi | MArea_MarkThisBlockLikeFree__FUi |
| 0x003f97b0 | 228 | GLOBAL |  | FMM_RFree__FUi | FMM_RFree__FUi |
| 0x003f98a0 | 64 | GLOBAL |  | FMM_IsLocked__FUi | FMM_IsLocked__FUi |
| 0x003f98e0 | 84 | GLOBAL |  | FMM_ELock_Dbg__FUi | FMM_ELock_Dbg__FUi |
| 0x003f9940 | 72 | GLOBAL |  | FMM_EUnlock_Dbg__FUi | FMM_EUnlock_Dbg__FUi |
| 0x003f9990 | 108 | GLOBAL |  | FMM_Lock__FUi | FMM_Lock__FUi |
| 0x003f9a00 | 288 | GLOBAL |  | FMM_Unlock__FUi | FMM_Unlock__FUi |
| 0x003f9b20 | 120 | GLOBAL |  | FMM_TempMallocUC__FUiUi | FMM_TempMallocUC__FUiUi |
| 0x003f9ba0 | 12 | GLOBAL |  | FMM_TempFree__FPv | FMM_TempFree__FPv |
| 0x003f9bb0 | 96 | GLOBAL |  | FMM_LnrBuff_Init__FUi | FMM_LnrBuff_Init__FUi |
| 0x003f9c10 | 40 | GLOBAL |  | FMM_LnrBuff_Close__Fv | FMM_LnrBuff_Close__Fv |
| 0x003f9c40 | 16 | GLOBAL |  | FMM_LnrBuff_Clear__Fv | FMM_LnrBuff_Clear__Fv |
| 0x003f9c50 | 184 | GLOBAL |  | FMM_LnrBuff_GetPointer___FUi | FMM_LnrBuff_GetPointer___FUi |
| 0x003f9d10 | 140 | GLOBAL |  | FMM_LnrBuff_GetPointer__FUi | FMM_LnrBuff_GetPointer__FUi |
| 0x003f9da0 | 16 | GLOBAL |  | FMM_LnrBuff_FreePointer__Fv | FMM_LnrBuff_FreePointer__Fv |
| 0x003f9db0 | 1704 | GLOBAL |  | LSI_PlayPSS__FPcb | LSI_PlayPSS__FPcb |
| 0x003fa460 | 8 | LOCAL |  | idleFn__Fi | idleFn__Fi |
| 0x003fa470 | 8 | LOCAL |  | fileInitFn__Fi | fileInitFn__Fi |
| 0x003fa480 | 52 | LOCAL |  | fileLoadFn__FPUcii | fileLoadFn__FPUcii |
| 0x003fa4c0 | 8 | GLOBAL |  | video_command__FiiiPc | video_command__FiiiPc |
| 0x003fa4d0 | 12 | GLOBAL |  | shellCoreSetIdleRoutine | shellCoreSetIdleRoutine |
| 0x003fa4e0 | 40 | GLOBAL |  | shellCoreCallIdleRoutine | shellCoreCallIdleRoutine |
| 0x003fa510 | 104 | GLOBAL |  | shellCoreAddIntcHandler | shellCoreAddIntcHandler |
| 0x003fa580 | 16 | GLOBAL |  | shellDisp_SetLSIDisplayOffset__Fii | shellDisp_SetLSIDisplayOffset__Fii |
| 0x003fa590 | 1256 | LOCAL |  | _SetDefDispEnv__FP13_ShellDblBuff | _SetDefDispEnv__FP13_ShellDblBuff |
| 0x003faa80 | 200 | LOCAL |  | _SetDispBuffers__FP13_ShellDblBuff | _SetDispBuffers__FP13_ShellDblBuff |
| 0x003fab50 | 84 | LOCAL |  | _CopyDrawEnvToContext2__FP17_ShellDispDrawEnv | _CopyDrawEnvToContext2__FP17_ShellDispDrawEnv |
| 0x003fabb0 | 672 | LOCAL |  | _SetDrawEnv__FP17_ShellDispDrawEnvssssss | _SetDrawEnv__FP17_ShellDispDrawEnvssssss |
| 0x003fae50 | 564 | LOCAL |  | _SetupShrinkSprites__FP13_ShellDblBuff | _SetupShrinkSprites__FP13_ShellDblBuff |
| 0x003fb090 | 1576 | LOCAL |  | _SetupClearBuffer__FP13_ShellDblBuff15EShellDispClear | _SetupClearBuffer__FP13_ShellDblBuff15EShellDispClear |
| 0x003fb6c0 | 248 | LOCAL |  | _shellDispResetGraph__F16EShellDispTVmode23EShellDispInterlaceMode | _shellDispResetGraph__F16EShellDispTVmode23EShellDispInterlaceMode |
| 0x003fb7c0 | 720 | LOCAL |  | _shellDispSetupDblBuffer__Fsssssss | _shellDispSetupDblBuffer__Fsssssss |
| 0x003fba90 | 132 | LOCAL |  | _VBlankHandler__FiPvPv | _VBlankHandler__FiPvPv |
| 0x003fbb20 | 112 | GLOBAL |  | shellDispRemoveVSyncCallback | shellDispRemoveVSyncCallback |
| 0x003fbb90 | 124 | GLOBAL |  | shellDispAddVSyncCallback | shellDispAddVSyncCallback |
| 0x003fbc10 | 388 | GLOBAL |  | shellDispInit | shellDispInit |
| 0x003fbda0 | 1268 | GLOBAL |  | shellDispSwapDblBuffer | shellDispSwapDblBuffer |
| 0x003fc2a0 | 24 | GLOBAL |  | shellDispSetClearType | shellDispSetClearType |
| 0x003fc2c0 | 20 | GLOBAL |  | shellDispSetTest | shellDispSetTest |
| 0x003fc2e0 | 92 | GLOBAL |  | shellDmaKickVif1List | shellDmaKickVif1List |
| 0x003fc340 | 80 | GLOBAL |  | shellDmaPrependVif1List | shellDmaPrependVif1List |
| 0x003fc390 | 120 | GLOBAL |  | shellDmaInit | shellDmaInit |
| 0x003fc410 | 84 | GLOBAL |  | shellDmaSetChannelOptions | shellDmaSetChannelOptions |
| 0x003fc470 | 112 | GLOBAL |  | shellDmaStart | shellDmaStart |
| 0x003fc4e0 | 120 | GLOBAL |  | shellDmaStartChain | shellDmaStartChain |
| 0x003fc560 | 136 | LOCAL |  | _FilenameToCdrom__FPCcPc | _FilenameToCdrom__FPCcPc |
| 0x003fc5f0 | 72 | LOCAL |  | _FilenameToHost0__FPCcPc | _FilenameToHost0__FPCcPc |
| 0x003fc640 | 396 | LOCAL |  | _GetIopImagePath__FPc | _GetIopImagePath__FPc |
| 0x003fc7d0 | 432 | LOCAL |  | _GetIopModulePath__FPCcPc | _GetIopModulePath__FPCcPc |
| 0x003fc980 | 56 | LOCAL |  | _NativeName__FPCcPc | _NativeName__FPCcPc |
| 0x003fc9c0 | 100 | LOCAL |  | _GetInfo__FPCcP9_FileInfo | _GetInfo__FPCcP9_FileInfo |
| 0x003fca30 | 324 | GLOBAL |  | shellFileInit | shellFileInit |
| 0x003fcb80 | 464 | GLOBAL |  | shellFileLoadStartModule | shellFileLoadStartModule |
| 0x003fcd50 | 120 | GLOBAL |  | shellFileStInit | shellFileStInit |
| 0x003fcdd0 | 184 | GLOBAL |  | shellFileStStart | shellFileStStart |
| 0x003fce90 | 148 | GLOBAL |  | shellFileStRead | shellFileStRead |
| 0x003fcf30 | 88 | GLOBAL |  | shellFileStStop | shellFileStStop |
| 0x003fcf90 | 12 | GLOBAL |  | MemAlign_Init | MemAlign_Init |
| 0x003fcfa0 | 20 | GLOBAL |  | MemAlign_Close | MemAlign_Close |
| 0x003fcfc0 | 96 | GLOBAL |  | MemAlign_Clear | MemAlign_Clear |
| 0x003fd020 | 108 | GLOBAL |  | MemAlign | MemAlign |
| 0x003fd090 | 128 | GLOBAL |  | shellMpegInit | shellMpegInit |
| 0x003fd110 | 104 | GLOBAL |  | shellMpegCreate | shellMpegCreate |
| 0x003fd180 | 728 | GLOBAL |  | shellMpegGetVideo | shellMpegGetVideo |
| 0x003fd460 | 104 | GLOBAL |  | shellMpegGetAudio | shellMpegGetAudio |
| 0x003fd4d0 | 92 | GLOBAL |  | shellMpegIsEnd | shellMpegIsEnd |
| 0x003fd530 | 8 | GLOBAL |  | shellMpegRecharge | shellMpegRecharge |
| 0x003fd540 | 1348 | LOCAL |  | _createCore__FiP15ShellMpegStream23EShellMpegMultiplexType19EShellMpegVideoType19EShellMpegAudioTypeiPFi_vPFPUcii_iiiUii | _createCore__FiP15ShellMpegStream23EShellMpegMultiplexType19EShellMpegVideoType19EShellMpegAudioTypeiPFi_vPFPUcii_iiiUii |
| 0x003fda90 | 140 | LOCAL |  | _mywaitipuidle__Fi | _mywaitipuidle__Fi |
| 0x003fdb20 | 224 | LOCAL |  | _installStrCallbacks__Fi | _installStrCallbacks__Fi |
| 0x003fdc00 | 364 | LOCAL |  | _videoCallback__FP7sceMpegP16sceMpegCbDataStrPv | _videoCallback__FP7sceMpegP16sceMpegCbDataStrPv |
| 0x003fdd70 | 456 | LOCAL |  | _audioCallback__FP7sceMpegP16sceMpegCbDataStrPv | _audioCallback__FP7sceMpegP16sceMpegCbDataStrPv |
| 0x003fdf40 | 316 | LOCAL |  | _nodataCallback__FP7sceMpegP13sceMpegCbDataPv | _nodataCallback__FP7sceMpegP13sceMpegCbDataPv |
| 0x003fe080 | 120 | LOCAL |  | _backgroundCallback__FP7sceMpegP13sceMpegCbDataPv | _backgroundCallback__FP7sceMpegP13sceMpegCbDataPv |
| 0x003fe100 | 8 | LOCAL |  | _errorCallback__FP7sceMpegP13sceMpegCbDataPv | _errorCallback__FP7sceMpegP13sceMpegCbDataPv |
| 0x003fe110 | 540 | LOCAL |  | _fillBuff__Fi | _fillBuff__Fi |
| 0x003fe330 | 52 | GLOBAL |  | setLoadImageDest__FP12_ShellDmaTagUiUi | setLoadImageDest__FP12_ShellDmaTagUiUi |
| 0x003fe370 | 444 | GLOBAL |  | setLoadImageTagsRows__FP12_ShellDmaTagPvUiUi | setLoadImageTagsRows__FP12_ShellDmaTagPvUiUi |
| 0x003fe530 | 400 | GLOBAL |  | setLoadImageTagsColumns__FP12_ShellDmaTagPvUiUi | setLoadImageTagsColumns__FP12_ShellDmaTagPvUiUi |
| 0x003fe6c0 | 2276 | GLOBAL |  | _VBlankHandler__Fi | _VBlankHandler__Fi |
| 0x003fefb0 | 88 | GLOBAL |  | CRTCM_SetDisplayOffset__Fff | CRTCM_SetDisplayOffset__Fff |
| 0x003ff010 | 1644 | GLOBAL |  | CRTCM_Init__FP14CRTCM_InitDesc | CRTCM_Init__FP14CRTCM_InitDesc |
| 0x003ff680 | 8 | GLOBAL |  | CRTCM_Close__Fv | CRTCM_Close__Fv |
| 0x003ff690 | 44 | GLOBAL |  | CRTCM_EnableDisplay__Fb | CRTCM_EnableDisplay__Fb |
| 0x003ff6c0 | 8 | GLOBAL |  | _CRTCM_DisplayEnabled__Fv | _CRTCM_DisplayEnabled__Fv |
| 0x003ff6d0 | 1092 | GLOBAL |  | CRTCM_GetAfterRenderPacket__Fv | CRTCM_GetAfterRenderPacket__Fv |
| 0x003ffb20 | 980 | GLOBAL |  | CRTCM_SwapBuffers__Fv | CRTCM_SwapBuffers__Fv |
| 0x003fff00 | 28 | GLOBAL |  | CRTCM_GetFrameReg__Fv | CRTCM_GetFrameReg__Fv |
| 0x003fff20 | 28 | GLOBAL |  | CRTCM_GetFrameReg_TVBuffer__Fv | CRTCM_GetFrameReg_TVBuffer__Fv |
| 0x003fff40 | 28 | GLOBAL |  | CRTCM_GetFrameReg_NextTVBuffer__Fv | CRTCM_GetFrameReg_NextTVBuffer__Fv |
| 0x003fff60 | 8 | GLOBAL |  | CRTCM_GetZBufReg__Fv | CRTCM_GetZBufReg__Fv |
| 0x003fff70 | 56 | GLOBAL |  | CRTCM_GetZBuffMAX__Fv | CRTCM_GetZBuffMAX__Fv |
| 0x003fffb0 | 8 | GLOBAL |  | CRTCM_GetFrameBufferPixelFormat__Fv | CRTCM_GetFrameBufferPixelFormat__Fv |
| 0x003fffc0 | 8 | GLOBAL |  | CRTCM_GetFrameBufferWidth__Fv | CRTCM_GetFrameBufferWidth__Fv |
| 0x003fffd0 | 8 | GLOBAL |  | CRTCM_GetFrameBufferHeight__Fv | CRTCM_GetFrameBufferHeight__Fv |
| 0x003fffe0 | 136 | GLOBAL |  | CRTCM_GetVRamPagesUsed__Fv | CRTCM_GetVRamPagesUsed__Fv |
| 0x00400070 | 108 | GLOBAL |  | iop_call__Fiiiii | iop_call__Fiiiii |
| 0x004000e0 | 124 | GLOBAL |  | iop_call3__Fiiiiiii | iop_call3__Fiiiiiii |
| 0x00400160 | 68 | GLOBAL |  | init_atable__Fv | init_atable__Fv |
| 0x004001b0 | 348 | GLOBAL |  | free_reg_sound__Fi | free_reg_sound__Fi |
| 0x00400310 | 264 | GLOBAL |  | status_reg_sound__Fi | status_reg_sound__Fi |
| 0x00400420 | 500 | GLOBAL |  | alloc_reg_sound__Fii | alloc_reg_sound__Fii |
| 0x00400620 | 84 | GLOBAL |  | find_reg_voice__Fi | find_reg_voice__Fi |
| 0x00400680 | 120 | GLOBAL |  | create_reg_voice__Fi | create_reg_voice__Fi |
| 0x00400700 | 156 | GLOBAL |  | clone_reg_voice__Fii | clone_reg_voice__Fii |
| 0x004007a0 | 288 | GLOBAL |  | delete_reg_voice__Fi | delete_reg_voice__Fi |
| 0x004008c0 | 92 | GLOBAL |  | find_reg_sound__Fi | find_reg_sound__Fi |
| 0x00400920 | 116 | GLOBAL |  | find_reg_sound2__Fi | find_reg_sound2__Fi |
| 0x004009a0 | 176 | GLOBAL |  | create_reg_sound__Fi | create_reg_sound__Fi |
| 0x00400a50 | 520 | GLOBAL |  | delete_reg_sound__Fi | delete_reg_sound__Fi |
| 0x00400c60 | 448 | GLOBAL |  | find_free_voice__Fi | find_free_voice__Fi |
| 0x00400e20 | 356 | GLOBAL |  | add_lock_tab__Fi | add_lock_tab__Fi |
| 0x00400f90 | 216 | GLOBAL |  | add_lock_tab_extra__Fiiiii | add_lock_tab_extra__Fiiiii |
| 0x00401070 | 1128 | GLOBAL |  | update_lock_sounds__Fv | update_lock_sounds__Fv |
| 0x004014e0 | 116 | GLOBAL |  | lock_mem__Fii | lock_mem__Fii |
| 0x00401560 | 112 | GLOBAL |  | lock_sound__Fi | lock_sound__Fi |
| 0x004015d0 | 236 | GLOBAL |  | load_file__Fi | load_file__Fi |
| 0x004016c0 | 744 | GLOBAL |  | play_voice__Fii | play_voice__Fii |
| 0x004019b0 | 140 | GLOBAL |  | lock_voice__Fii | lock_voice__Fii |
| 0x00401a40 | 492 | GLOBAL |  | stop_voice__Fii | stop_voice__Fii |
| 0x00401c30 | 144 | GLOBAL |  | volume_voice__Fii | volume_voice__Fii |
| 0x00401cc0 | 288 | GLOBAL |  | pitch_voice__Fii | pitch_voice__Fii |
| 0x00401de0 | 248 | GLOBAL |  | get_status_voice__Fi | get_status_voice__Fi |
| 0x00401ee0 | 448 | GLOBAL |  | get_pos_voice__Fi | get_pos_voice__Fi |
| 0x004020a0 | 72 | GLOBAL |  | get_voice_size__Fi | get_voice_size__Fi |
| 0x004020f0 | 288 | GLOBAL |  | set_pos_voice__Fii | set_pos_voice__Fii |
| 0x00402210 | 1520 | GLOBAL |  | update_voices__Fv | update_voices__Fv |
| 0x00402800 | 116 | GLOBAL |  | kill_engine__Fv | kill_engine__Fv |
| 0x00402880 | 60 | GLOBAL |  | LSI_WaitForLoad_SE__Fv | LSI_WaitForLoad_SE__Fv |
| 0x004028c0 | 856 | GLOBAL |  | sound_command__FiiiPc | sound_command__FiiiPc |
| 0x00402c20 | 580 | GLOBAL |  | test_file__FPc | test_file__FPc |
| 0x00402e70 | 676 | GLOBAL |  | init_sound__Fv | init_sound__Fv |
| 0x00403120 | 356 | GLOBAL | LSISoundBuffer | QueryInterface | QueryInterface__14LSISoundBufferFPUiPPv |
| 0x00403290 | 168 | GLOBAL | LSISoundBuffer | Release | Release__14LSISoundBufferFv |
| 0x00403340 | 200 | GLOBAL | LSISoundBuffer | GetCurrentPosition | GetCurrentPosition__14LSISoundBufferFPUiPUi |
| 0x00403410 | 172 | GLOBAL | LSISoundBuffer | GetStatus | GetStatus__14LSISoundBufferFPUi |
| 0x004034c0 | 132 | GLOBAL | LSISoundBuffer | Lock | Lock__14LSISoundBufferFUiUiPPvPUiPPvPUiUi |
| 0x00403550 | 180 | GLOBAL | LSISoundBuffer | Play | Play__14LSISoundBufferFUiUiUi |
| 0x00403610 | 180 | GLOBAL | LSISoundBuffer | SetCurrentPosition | SetCurrentPosition__14LSISoundBufferFUi |
| 0x004036d0 | 120 | GLOBAL | LSISoundBuffer | SetFormat | SetFormat__14LSISoundBufferFPC13tWAVEFORMATEX |
| 0x00403750 | 180 | GLOBAL | LSISoundBuffer | SetVolume | SetVolume__14LSISoundBufferFi |
| 0x00403810 | 156 | GLOBAL | LSISoundBuffer | SetPan | SetPan__14LSISoundBufferFi |
| 0x004038b0 | 236 | GLOBAL | LSISoundBuffer | SetFrequency | SetFrequency__14LSISoundBufferFUi |
| 0x004039a0 | 132 | GLOBAL | LSISoundBuffer | Stop | Stop__14LSISoundBufferFv |
| 0x00403a30 | 140 | GLOBAL | LSISoundBuffer | Unlock | Unlock__14LSISoundBufferFPvUiPvUi |
| 0x00403ac0 | 228 | GLOBAL | LSISound3DListener | {ctor} | __ct__18LSISound3DListenerFv |
| 0x00403bb0 | 140 | GLOBAL | LSISound3DListener | Release | Release__18LSISound3DListenerFv |
| 0x00403c40 | 192 | GLOBAL | LSISound3DListener | GetAllParameters | GetAllParameters__18LSISound3DListenerFP15_LSIS3DLISTENER |
| 0x00403d00 | 192 | GLOBAL | LSISound3DListener | SetAllParameters | SetAllParameters__18LSISound3DListenerFPC15_LSIS3DLISTENERUi |
| 0x00403dc0 | 148 | GLOBAL | LSISound | Release | Release__8LSISoundFv |
| 0x00403e60 | 248 | GLOBAL | LSISound | CreateSoundBuffer | CreateSoundBuffer__8LSISoundFPC15_LSISBUFFERDESCPP15ILSISoundBufferPv |
| 0x00403f60 | 276 | GLOBAL | LSISound | GetCaps | GetCaps__8LSISoundFP9_LSISCAPS |
| 0x00404080 | 232 | GLOBAL | LSISound | DuplicateSoundBuffer | DuplicateSoundBuffer__8LSISoundFP15ILSISoundBufferPP15ILSISoundBuffer |
| 0x00404170 | 128 | GLOBAL | LSISound | SetCooperativeLevel | SetCooperativeLevel__8LSISoundFPvUi |
| 0x004041f0 | 128 | GLOBAL | LSISound | SetSpeakerConfig | SetSpeakerConfig__8LSISoundFUi |
| 0x00404270 | 156 | LOCAL |  | pad_button_read__FP9TSLOTDATA | pad_button_read__FP9TSLOTDATA |
| 0x00404310 | 992 | LOCAL |  | attach_controller__FP9TSLOTDATA | attach_controller__FP9TSLOTDATA |
| 0x004046f0 | 152 | LOCAL |  | pad_port_open__FiiP9TSLOTDATA | pad_port_open__FiiP9TSLOTDATA |
| 0x00404790 | 72 | GLOBAL | LSIPad | {ctor} | __ct__6LSIPadFv |
| 0x004047e0 | 96 | GLOBAL | LSIPad | {dtor} | __dt__6LSIPadFv |
| 0x00404840 | 12 | GLOBAL | LSIPad | Update | Update__6LSIPadFv |
| 0x00404850 | 68 | GLOBAL | LSIPad | GetButton0 | GetButton0__6LSIPadFi |
| 0x004048a0 | 128 | GLOBAL |  | TexMemMan_Init__FUiUi | TexMemMan_Init__FUiUi |
| 0x00404920 | 32 | GLOBAL |  | TexMemMan_Close__Fv | TexMemMan_Close__Fv |
| 0x00404940 | 180 | GLOBAL |  | TexMemMan_PlaceTexture__FUiP11TMM_TEXINFO | TexMemMan_PlaceTexture__FUiP11TMM_TEXINFO |
| 0x00404a00 | 80 | GLOBAL |  | TexMemMan_IsInVRam_s0__FP11TMM_TEXINFO | TexMemMan_IsInVRam_s0__FP11TMM_TEXINFO |
| 0x00404a50 | 80 | GLOBAL |  | TexMemMan_IsInVRam_s1__FP11TMM_TEXINFO | TexMemMan_IsInVRam_s1__FP11TMM_TEXINFO |
| 0x00404aa0 | 28 | GLOBAL |  | TexMemMan_InitTexCtrlBuff__Fv | TexMemMan_InitTexCtrlBuff__Fv |
| 0x00404ac0 | 128 | GLOBAL |  | TexMemMan_AddSubPacketReq__FP1 | TexMemMan_AddSubPacketReq__FP1 |
| 0x00404b40 | 768 | GLOBAL |  | TexMemMan_ProcessTexCtrlBuff___Fv | TexMemMan_ProcessTexCtrlBuff___Fv |
| 0x00404e40 | 156 | GLOBAL |  | TexMemMan_EndPacketReq__Fv | TexMemMan_EndPacketReq__Fv |
| 0x00404ee0 | 176 | GLOBAL |  | TexMemMan_ModifyMemoryReq2__FPUsUs | TexMemMan_ModifyMemoryReq2__FPUsUs |
| 0x00404f90 | 92 | GLOBAL |  | GifMan_DmaSendToGif__FUi | GifMan_DmaSendToGif__FUi |
| 0x00405000 | 16 | GLOBAL |  | RFU000_FullReset | RFU000_FullReset |
| 0x00405010 | 16 | GLOBAL |  | ResetEE | ResetEE |
| 0x00405020 | 16 | GLOBAL |  | SetGsCrt | SetGsCrt |
| 0x00405030 | 16 | GLOBAL |  | RFU003 | RFU003 |
| 0x00405040 | 16 | GLOBAL |  | _Exit | _Exit |
| 0x00405050 | 16 | GLOBAL |  | RFU005 | RFU005 |
| 0x00405060 | 16 | GLOBAL |  | _LoadExecPS2 | _LoadExecPS2 |
| 0x00405070 | 16 | GLOBAL |  | _ExecPS2 | _ExecPS2 |
| 0x00405080 | 16 | GLOBAL |  | RFU008 | RFU008 |
| 0x00405090 | 16 | GLOBAL |  | RFU009 | RFU009 |
| 0x004050a0 | 16 | GLOBAL |  | AddSbusIntcHandler | AddSbusIntcHandler |
| 0x004050b0 | 16 | GLOBAL |  | RemoveSbusIntcHandler | RemoveSbusIntcHandler |
| 0x004050c0 | 16 | GLOBAL |  | Interrupt2Iop | Interrupt2Iop |
| 0x004050d0 | 16 | GLOBAL |  | SetVTLBRefillHandler | SetVTLBRefillHandler |
| 0x004050e0 | 16 | GLOBAL |  | SetVCommonHandler | SetVCommonHandler |
| 0x004050f0 | 16 | GLOBAL |  | SetVInterruptHandler | SetVInterruptHandler |
| 0x00405100 | 16 | GLOBAL |  | AddIntcHandler | AddIntcHandler |
| 0x00405110 | 16 | GLOBAL |  | AddIntcHandler2 | AddIntcHandler2 |
| 0x00405120 | 16 | GLOBAL |  | RemoveIntcHandler | RemoveIntcHandler |
| 0x00405130 | 16 | GLOBAL |  | AddDmacHandler | AddDmacHandler |
| 0x00405140 | 16 | GLOBAL |  | AddDmacHandler2 | AddDmacHandler2 |
| 0x00405150 | 16 | GLOBAL |  | RemoveDmacHandler | RemoveDmacHandler |
| 0x00405160 | 16 | GLOBAL |  | _EnableIntc | _EnableIntc |
| 0x00405170 | 16 | GLOBAL |  | _DisableIntc | _DisableIntc |
| 0x00405180 | 16 | GLOBAL |  | _EnableDmac | _EnableDmac |
| 0x00405190 | 16 | GLOBAL |  | _DisableDmac | _DisableDmac |
| 0x004051a0 | 16 | GLOBAL |  | SetAlarm | SetAlarm |
| 0x004051b0 | 16 | GLOBAL |  | ReleaseAlarm | ReleaseAlarm |
| 0x004051c0 | 16 | GLOBAL |  | _iEnableIntc | _iEnableIntc |
| 0x004051d0 | 16 | GLOBAL |  | _iDisableIntc | _iDisableIntc |
| 0x004051e0 | 16 | GLOBAL |  | _iEnableDmac | _iEnableDmac |
| 0x004051f0 | 16 | GLOBAL |  | _iDisableDmac | _iDisableDmac |
| 0x00405200 | 16 | GLOBAL |  | iSetAlarm | iSetAlarm |
| 0x00405210 | 16 | GLOBAL |  | iReleaseAlarm | iReleaseAlarm |
| 0x00405220 | 16 | GLOBAL |  | CreateThread | CreateThread |
| 0x00405230 | 16 | GLOBAL |  | DeleteThread | DeleteThread |
| 0x00405240 | 16 | GLOBAL |  | StartThread | StartThread |
| 0x00405250 | 16 | GLOBAL |  | ExitThread | ExitThread |
| 0x00405260 | 16 | GLOBAL |  | ExitDeleteThread | ExitDeleteThread |
| 0x00405270 | 16 | GLOBAL |  | TerminateThread | TerminateThread |
| 0x00405280 | 16 | GLOBAL |  | iTerminateThread | iTerminateThread |
| 0x00405290 | 16 | GLOBAL |  | DisableDispatchThread | DisableDispatchThread |
| 0x004052a0 | 16 | GLOBAL |  | EnableDispatchThread | EnableDispatchThread |
| 0x004052b0 | 16 | GLOBAL |  | ChangeThreadPriority | ChangeThreadPriority |
| 0x004052c0 | 16 | GLOBAL |  | iChangeThreadPriority | iChangeThreadPriority |
| 0x004052d0 | 16 | GLOBAL |  | RotateThreadReadyQueue | RotateThreadReadyQueue |
| 0x004052e0 | 16 | GLOBAL |  | _iRotateThreadReadyQueue | _iRotateThreadReadyQueue |
| 0x004052f0 | 16 | GLOBAL |  | ReleaseWaitThread | ReleaseWaitThread |
| 0x00405300 | 16 | GLOBAL |  | iReleaseWaitThread | iReleaseWaitThread |
| 0x00405310 | 16 | GLOBAL |  | GetThreadId | GetThreadId |
| 0x00405320 | 16 | GLOBAL |  | ReferThreadStatus | ReferThreadStatus |
| 0x00405330 | 16 | GLOBAL |  | iReferThreadStatus | iReferThreadStatus |
| 0x00405340 | 16 | GLOBAL |  | SleepThread | SleepThread |
| 0x00405350 | 16 | GLOBAL |  | WakeupThread | WakeupThread |
| 0x00405360 | 16 | GLOBAL |  | _iWakeupThread | _iWakeupThread |
| 0x00405370 | 16 | GLOBAL |  | CancelWakeupThread | CancelWakeupThread |
| 0x00405380 | 16 | GLOBAL |  | iCancelWakeupThread | iCancelWakeupThread |
| 0x00405390 | 16 | GLOBAL |  | SuspendThread | SuspendThread |
| 0x004053a0 | 16 | GLOBAL |  | _iSuspendThread | _iSuspendThread |
| 0x004053b0 | 16 | GLOBAL |  | ResumeThread | ResumeThread |
| 0x004053c0 | 16 | GLOBAL |  | iResumeThread | iResumeThread |
| 0x004053d0 | 16 | GLOBAL |  | JoinThread | JoinThread |
| 0x004053e0 | 16 | GLOBAL |  | RFU060 | RFU060 |
| 0x004053f0 | 16 | GLOBAL |  | RFU061 | RFU061 |
| 0x00405400 | 16 | GLOBAL |  | EndOfHeap | EndOfHeap |
| 0x00405410 | 16 | GLOBAL |  | RFU063 | RFU063 |
| 0x00405420 | 16 | GLOBAL |  | CreateSema | CreateSema |
| 0x00405430 | 16 | GLOBAL |  | DeleteSema | DeleteSema |
| 0x00405440 | 16 | GLOBAL |  | SignalSema | SignalSema |
| 0x00405450 | 16 | GLOBAL |  | iSignalSema | iSignalSema |
| 0x00405460 | 16 | GLOBAL |  | WaitSema | WaitSema |
| 0x00405470 | 16 | GLOBAL |  | PollSema | PollSema |
| 0x00405480 | 16 | GLOBAL |  | iPollSema | iPollSema |
| 0x00405490 | 16 | GLOBAL |  | ReferSemaStatus | ReferSemaStatus |
| 0x004054a0 | 16 | GLOBAL |  | iReferSemaStatus | iReferSemaStatus |
| 0x004054b0 | 16 | GLOBAL |  | RFU073 | RFU073 |
| 0x004054c0 | 16 | GLOBAL |  | SetOsdConfigParam | SetOsdConfigParam |
| 0x004054d0 | 16 | GLOBAL |  | GetOsdConfigParam | GetOsdConfigParam |
| 0x004054e0 | 16 | GLOBAL |  | GetGsHParam | GetGsHParam |
| 0x004054f0 | 16 | GLOBAL |  | GetGsVParam | GetGsVParam |
| 0x00405500 | 16 | GLOBAL |  | SetGsHParam | SetGsHParam |
| 0x00405510 | 16 | GLOBAL |  | SetGsVParam | SetGsVParam |
| 0x00405520 | 16 | GLOBAL |  | RFU080_CreateEventFlag | RFU080_CreateEventFlag |
| 0x00405530 | 16 | GLOBAL |  | RFU081_DeleteEventFlag | RFU081_DeleteEventFlag |
| 0x00405540 | 16 | GLOBAL |  | RFU082_SetEventFlag | RFU082_SetEventFlag |
| 0x00405550 | 16 | GLOBAL |  | RFU083_iSetEventFlag | RFU083_iSetEventFlag |
| 0x00405560 | 16 | GLOBAL |  | RFU084_ClearEventFlag | RFU084_ClearEventFlag |
| 0x00405570 | 16 | GLOBAL |  | RFU085_iClearEventFlag | RFU085_iClearEventFlag |
| 0x00405580 | 16 | GLOBAL |  | RFU086_WaitEvnetFlag | RFU086_WaitEvnetFlag |
| 0x00405590 | 16 | GLOBAL |  | RFU087_PollEvnetFlag | RFU087_PollEvnetFlag |
| 0x004055a0 | 16 | GLOBAL |  | RFU088_iPollEvnetFlag | RFU088_iPollEvnetFlag |
| 0x004055b0 | 16 | GLOBAL |  | RFU089_ReferEventFlagStatus | RFU089_ReferEventFlagStatus |
| 0x004055c0 | 16 | GLOBAL |  | RFU090_iReferEventFlagStatus | RFU090_iReferEventFlagStatus |
| 0x004055d0 | 16 | GLOBAL |  | RFU091 | RFU091 |
| 0x004055e0 | 16 | GLOBAL |  | EnableIntcHandler | EnableIntcHandler |
| 0x004055f0 | 16 | GLOBAL |  | iEnableIntcHandler | iEnableIntcHandler |
| 0x00405600 | 16 | GLOBAL |  | DisableIntcHandler | DisableIntcHandler |
| 0x00405610 | 16 | GLOBAL |  | iDisableIntcHandler | iDisableIntcHandler |
| 0x00405620 | 16 | GLOBAL |  | EnableDmacHandler | EnableDmacHandler |
| 0x00405630 | 16 | GLOBAL |  | iEnableDmacHandler | iEnableDmacHandler |
| 0x00405640 | 16 | GLOBAL |  | DisableDmacHandler | DisableDmacHandler |
| 0x00405650 | 16 | GLOBAL |  | iDisableDmacHandler | iDisableDmacHandler |
| 0x00405660 | 16 | GLOBAL |  | KSeg0 | KSeg0 |
| 0x00405670 | 16 | GLOBAL |  | EnableCache | EnableCache |
| 0x00405680 | 16 | GLOBAL |  | DisableCache | DisableCache |
| 0x00405690 | 16 | GLOBAL |  | GetCop0 | GetCop0 |
| 0x004056a0 | 16 | GLOBAL |  | FlushCache | FlushCache |
| 0x004056b0 | 16 | GLOBAL |  | CpuConfig | CpuConfig |
| 0x004056c0 | 16 | GLOBAL |  | iGetCop0 | iGetCop0 |
| 0x004056d0 | 16 | GLOBAL |  | iFlushCache | iFlushCache |
| 0x004056e0 | 16 | GLOBAL |  | iCpuConfig | iCpuConfig |
| 0x004056f0 | 16 | GLOBAL |  | sceSifStopDma | sceSifStopDma |
| 0x00405700 | 16 | GLOBAL |  | SetCPUTimerHandler | SetCPUTimerHandler |
| 0x00405710 | 16 | GLOBAL |  | SetCPUTimer | SetCPUTimer |
| 0x00405720 | 16 | GLOBAL |  | SetOsdConfigParam2 | SetOsdConfigParam2 |
| 0x00405730 | 16 | GLOBAL |  | GetOsdConfigParam2 | GetOsdConfigParam2 |
| 0x00405740 | 16 | GLOBAL |  | GsGetIMR | GsGetIMR |
| 0x00405750 | 16 | GLOBAL |  | iGsGetIMR | iGsGetIMR |
| 0x00405760 | 16 | GLOBAL |  | GsPutIMR | GsPutIMR |
| 0x00405770 | 16 | GLOBAL |  | iGsPutIMR | iGsPutIMR |
| 0x00405780 | 16 | GLOBAL |  | SetPgifHandler | SetPgifHandler |
| 0x00405790 | 16 | GLOBAL |  | SetVSyncFlag | SetVSyncFlag |
| 0x004057a0 | 16 | GLOBAL |  | RFU116 | RFU116 |
| 0x004057b0 | 16 | GLOBAL |  | _print | _print |
| 0x004057c0 | 16 | GLOBAL |  | sceSifDmaStat | sceSifDmaStat |
| 0x004057d0 | 16 | GLOBAL |  | isceSifDmaStat | isceSifDmaStat |
| 0x004057e0 | 16 | GLOBAL |  | sceSifSetDma | sceSifSetDma |
| 0x004057f0 | 16 | GLOBAL |  | isceSifSetDma | isceSifSetDma |
| 0x00405800 | 16 | GLOBAL |  | sceSifSetDChain | sceSifSetDChain |
| 0x00405810 | 16 | GLOBAL |  | isceSifSetDChain | isceSifSetDChain |
| 0x00405820 | 16 | GLOBAL |  | sceSifSetReg | sceSifSetReg |
| 0x00405830 | 16 | GLOBAL |  | sceSifGetReg | sceSifGetReg |
| 0x00405840 | 16 | GLOBAL |  | _ExecOSD | _ExecOSD |
| 0x00405850 | 16 | GLOBAL |  | Deci2Call | Deci2Call |
| 0x00405860 | 16 | GLOBAL |  | PSMode | PSMode |
| 0x00405870 | 16 | GLOBAL |  | MachineType | MachineType |
| 0x00405880 | 16 | GLOBAL |  | GetMemorySize | GetMemorySize |
| 0x00405890 | 16 | GLOBAL |  | _InitTLB | _InitTLB |
| 0x004058a0 | 12 | GLOBAL |  | sceResetttyinit | sceResetttyinit |
| 0x004058b0 | 144 | GLOBAL |  | VSync | VSync |
| 0x00405940 | 164 | GLOBAL |  | VSync2 | VSync2 |
| 0x00405b10 | 8 | GLOBAL |  | ioctl | ioctl |
| 0x00405c70 | 164 | GLOBAL |  | _sceSDC | _sceSDC |
| 0x00405d18 | 124 | GLOBAL |  | SyncDCache | SyncDCache |
| 0x00405d98 | 20 | GLOBAL |  | iSyncDCache | iSyncDCache |
| 0x00405db0 | 164 | GLOBAL |  | _sceIDC | _sceIDC |
| 0x00405e58 | 124 | GLOBAL |  | InvalidDCache | InvalidDCache |
| 0x00405ed8 | 20 | GLOBAL |  | iInvalidDCache | iInvalidDCache |
| 0x00405ef0 | 16 | GLOBAL |  | QueryIntrContext | QueryIntrContext |
| 0x00405f00 | 104 | GLOBAL |  | DisableIntc | DisableIntc |
| 0x00405f68 | 104 | GLOBAL |  | EnableIntc | EnableIntc |
| 0x00405fd0 | 104 | GLOBAL |  | DisableDmac | DisableDmac |
| 0x00406038 | 104 | GLOBAL |  | EnableDmac | EnableDmac |
| 0x004060a0 | 32 | GLOBAL |  | iEnableIntc | iEnableIntc |
| 0x004060c0 | 32 | GLOBAL |  | iDisableIntc | iDisableIntc |
| 0x004060e0 | 32 | GLOBAL |  | iEnableDmac | iEnableDmac |
| 0x00406100 | 32 | GLOBAL |  | iDisableDmac | iDisableDmac |
| 0x00406120 | 216 | LOCAL |  | topThread | topThread |
| 0x004061f8 | 212 | GLOBAL |  | InitThread | InitThread |
| 0x004062d0 | 148 | GLOBAL |  | iWakeupThread | iWakeupThread |
| 0x00406368 | 124 | GLOBAL |  | iRotateThreadReadyQueue | iRotateThreadReadyQueue |
| 0x004063e8 | 152 | GLOBAL |  | iSuspendThread | iSuspendThread |
| 0x00406480 | 68 | GLOBAL |  | sceDeci2Open | sceDeci2Open |
| 0x004064c8 | 36 | GLOBAL |  | sceDeci2Close | sceDeci2Close |
| 0x004064f0 | 48 | GLOBAL |  | sceDeci2ReqSend | sceDeci2ReqSend |
| 0x00406520 | 36 | GLOBAL |  | sceDeci2Poll | sceDeci2Poll |
| 0x00406548 | 52 | GLOBAL |  | sceDeci2ExRecv | sceDeci2ExRecv |
| 0x00406580 | 52 | GLOBAL |  | sceDeci2ExSend | sceDeci2ExSend |
| 0x004065b8 | 48 | GLOBAL |  | sceDeci2ExReqSend | sceDeci2ExReqSend |
| 0x004065e8 | 36 | GLOBAL |  | sceDeci2ExLock | sceDeci2ExLock |
| 0x00406610 | 36 | GLOBAL |  | sceDeci2ExUnLock | sceDeci2ExUnLock |
| 0x00406638 | 36 | GLOBAL |  | kputs | kputs |
| 0x00406660 | 36 | LOCAL |  | QueueInit | QueueInit |
| 0x00406688 | 60 | LOCAL |  | QueuePeekWriteDone | QueuePeekWriteDone |
| 0x004066c8 | 60 | LOCAL |  | QueuePeekReadDone | QueuePeekReadDone |
| 0x00406708 | 404 | LOCAL |  | sceTtyHandler | sceTtyHandler |
| 0x004068a0 | 336 | GLOBAL |  | sceTtyWrite | sceTtyWrite |
| 0x004069f0 | 208 | GLOBAL |  | sceTtyRead | sceTtyRead |
| 0x00406ac0 | 188 | GLOBAL |  | sceTtyInit | sceTtyInit |
| 0x00406b80 | 56 | LOCAL |  | kputchar | kputchar |
| 0x00406bb8 | 176 | LOCAL |  | deci2Putchar | deci2Putchar |
| 0x00406c68 | 52 | LOCAL |  | serialPutchar | serialPutchar |
| 0x00406ca0 | 144 | LOCAL |  | ftoi | ftoi |
| 0x00406d30 | 360 | LOCAL |  | printfloat | printfloat |
| 0x00406e98 | 1516 | GLOBAL |  | _printf | _printf |
| 0x00407488 | 56 | GLOBAL |  | kprintf | kprintf |
| 0x004074c0 | 96 | GLOBAL |  | scePrintf | scePrintf |
| 0x00407520 | 28 | LOCAL |  | _set_sreg | _set_sreg |
| 0x00407540 | 12 | LOCAL |  | _change_addr | _change_addr |
| 0x00407550 | 24 | GLOBAL |  | sceSifGetSreg | sceSifGetSreg |
| 0x00407568 | 28 | GLOBAL |  | sceSifSetSreg | sceSifSetSreg |
| 0x00407588 | 12 | GLOBAL |  | sceSifGetDataTable | sceSifGetDataTable |
| 0x00407598 | 640 | GLOBAL |  | sceSifInitCmd | sceSifInitCmd |
| 0x00407818 | 52 | GLOBAL |  | sceSifExitCmd | sceSifExitCmd |
| 0x00407850 | 24 | GLOBAL |  | sceSifSetCmdBuffer | sceSifSetCmdBuffer |
| 0x00407868 | 24 | GLOBAL |  | sceSifSetSysCmdBuffer | sceSifSetSysCmdBuffer |
| 0x00407880 | 44 | GLOBAL |  | sceSifAddCmdHandler | sceSifAddCmdHandler |
| 0x004078b0 | 40 | GLOBAL |  | sceSifRemoveCmdHandler | sceSifRemoveCmdHandler |
| 0x004078d8 | 308 | LOCAL |  | _sceSifSendCmd | _sceSifSendCmd |
| 0x00407a10 | 60 | GLOBAL |  | sceSifSendCmd | sceSifSendCmd |
| 0x00407a50 | 60 | GLOBAL |  | isceSifSendCmd | isceSifSendCmd |
| 0x00407a90 | 296 | LOCAL |  | _sceSifCmdIntrHdlr | _sceSifCmdIntrHdlr |
| 0x00407bb8 | 176 | GLOBAL |  | sceSifWriteBackDCache | sceSifWriteBackDCache |
| 0x00407c68 | 412 | GLOBAL |  | sceSifInitRpc | sceSifInitRpc |
| 0x00407e08 | 36 | GLOBAL |  | sceSifExitRpc | sceSifExitRpc |
| 0x00407e30 | 168 | LOCAL |  | _sceRpcGetPacket | _sceRpcGetPacket |
| 0x00407ed8 | 28 | LOCAL |  | _sceRpcFreePacket | _sceRpcFreePacket |
| 0x00407ef8 | 48 | LOCAL |  | _sceRpcGetFPacket | _sceRpcGetFPacket |
| 0x00407f28 | 64 | LOCAL |  | _sceRpcGetFPacket2 | _sceRpcGetFPacket2 |
| 0x00407f68 | 180 | LOCAL |  | _request_end | _request_end |
| 0x00408020 | 96 | LOCAL |  | _alarm_rdata | _alarm_rdata |
| 0x00408080 | 208 | LOCAL |  | _request_rdata | _request_rdata |
| 0x00408150 | 344 | GLOBAL |  | sceSifGetOtherData | sceSifGetOtherData |
| 0x004082a8 | 76 | LOCAL |  | _search_svdata | _search_svdata |
| 0x004082f8 | 176 | LOCAL |  | _request_bind | _request_bind |
| 0x004083a8 | 320 | GLOBAL |  | sceSifBindRpc | sceSifBindRpc |
| 0x004084e8 | 140 | LOCAL |  | _request_call | _request_call |
| 0x00408578 | 492 | GLOBAL |  | sceSifCallRpc | sceSifCallRpc |
| 0x00408768 | 60 | GLOBAL |  | sceSifCheckStatRpc | sceSifCheckStatRpc |
| 0x004087a8 | 148 | GLOBAL |  | sceSifSetRpcQueue | sceSifSetRpcQueue |
| 0x00408840 | 208 | GLOBAL |  | sceSifRegisterRpc | sceSifRegisterRpc |
| 0x00408910 | 148 | GLOBAL |  | sceSifRemoveRpc | sceSifRemoveRpc |
| 0x004089a8 | 144 | GLOBAL |  | sceSifRemoveRpcQueue | sceSifRemoveRpcQueue |
| 0x00408a38 | 88 | GLOBAL |  | sceSifGetNextRequest | sceSifGetNextRequest |
| 0x00408a90 | 452 | GLOBAL |  | sceSifExecRequest | sceSifExecRequest |
| 0x00408c58 | 64 | GLOBAL |  | sceSifRpcLoop | sceSifRpcLoop |
| 0x00408c98 | 92 | LOCAL |  | _sceFsIobSemaMK | _sceFsIobSemaMK |
| 0x00408cf8 | 136 | LOCAL |  | new_iob | new_iob |
| 0x00408d80 | 108 | LOCAL |  | get_iob | get_iob |
| 0x00408df0 | 932 | LOCAL |  | _sceFs_Rcv_Intr | _sceFs_Rcv_Intr |
| 0x00409198 | 76 | LOCAL |  | _sceFsSemInit | _sceFsSemInit |
| 0x004091e8 | 44 | LOCAL |  | _sceFsWaitS | _sceFsWaitS |
| 0x00409218 | 12 | LOCAL |  | _sceFsSigSema | _sceFsSigSema |
| 0x00409228 | 152 | GLOBAL |  | scePowerOffHandler | scePowerOffHandler |
| 0x004092c0 | 44 | LOCAL |  | _sceFs_Poff_Intr | _sceFs_Poff_Intr |
| 0x004092f0 | 516 | GLOBAL |  | sceFsInit | sceFsInit |
| 0x004094f8 | 140 | LOCAL |  | _fs_version | _fs_version |
| 0x00409588 | 52 | GLOBAL |  | sceFsReset | sceFsReset |
| 0x004095c0 | 644 | GLOBAL |  | sceOpen | sceOpen |
| 0x00409848 | 380 | GLOBAL |  | sceClose | sceClose |
| 0x004099c8 | 568 | GLOBAL |  | sceLseek | sceLseek |
| 0x00409c00 | 608 | GLOBAL |  | sceRead | sceRead |
| 0x00409e60 | 704 | GLOBAL |  | sceWrite | sceWrite |
| 0x0040a120 | 884 | GLOBAL |  | sceIoctl | sceIoctl |
| 0x0040a498 | 476 | GLOBAL |  | sceIoctl2 | sceIoctl2 |
| 0x0040a678 | 420 | LOCAL |  | _sceCallCode | _sceCallCode |
| 0x0040a820 | 28 | GLOBAL |  | sceRemove | sceRemove |
| 0x0040a840 | 428 | GLOBAL |  | sceMkdir | sceMkdir |
| 0x0040a9f0 | 28 | GLOBAL |  | sceRmdir | sceRmdir |
| 0x0040aa10 | 620 | GLOBAL |  | sceFormat | sceFormat |
| 0x0040ac80 | 288 | GLOBAL |  | sceAddDrv | sceAddDrv |
| 0x0040ada0 | 28 | GLOBAL |  | sceDelDrv | sceDelDrv |
| 0x0040adc0 | 200 | GLOBAL |  | sceDopen | sceDopen |
| 0x0040ae88 | 360 | GLOBAL |  | sceDclose | sceDclose |
| 0x0040aff0 | 344 | GLOBAL |  | sceDread | sceDread |
| 0x0040b148 | 416 | GLOBAL |  | sceGetstat | sceGetstat |
| 0x0040b2e8 | 572 | GLOBAL |  | sceChstat | sceChstat |
| 0x0040b528 | 496 | GLOBAL |  | sceRename | sceRename |
| 0x0040b718 | 28 | GLOBAL |  | sceChdir | sceChdir |
| 0x0040b738 | 408 | GLOBAL |  | sceSync | sceSync |
| 0x0040b8d0 | 620 | GLOBAL |  | sceMount | sceMount |
| 0x0040bb40 | 28 | GLOBAL |  | sceUmount | sceUmount |
| 0x0040bb60 | 568 | GLOBAL |  | sceLseek64 | sceLseek64 |
| 0x0040bd98 | 568 | GLOBAL |  | sceDevctl | sceDevctl |
| 0x0040bfd0 | 480 | GLOBAL |  | sceSymlink | sceSymlink |
| 0x0040c1b0 | 456 | GLOBAL |  | sceReadlink | sceReadlink |
| 0x0040c378 | 136 | GLOBAL |  | sceSifInitIopHeap | sceSifInitIopHeap |
| 0x0040c400 | 112 | GLOBAL |  | sceSifAllocIopHeap | sceSifAllocIopHeap |
| 0x0040c470 | 128 | GLOBAL |  | sceSifAllocSysMemory | sceSifAllocSysMemory |
| 0x0040c4f0 | 120 | GLOBAL |  | sceSifFreeSysMemory | sceSifFreeSysMemory |
| 0x0040c568 | 28 | GLOBAL |  | sceSifFreeIopHeap | sceSifFreeIopHeap |
| 0x0040c588 | 240 | GLOBAL |  | sceSifLoadIopHeap | sceSifLoadIopHeap |
| 0x0040c678 | 112 | GLOBAL |  | sceSifQueryMemSize | sceSifQueryMemSize |
| 0x0040c6e8 | 112 | GLOBAL |  | sceSifQueryMaxFreeMemSize | sceSifQueryMaxFreeMemSize |
| 0x0040c758 | 112 | GLOBAL |  | sceSifQueryTotalFreeMemSize | sceSifQueryTotalFreeMemSize |
| 0x0040c7c8 | 124 | GLOBAL |  | sceSifQueryBlockTopAddress | sceSifQueryBlockTopAddress |
| 0x0040c848 | 124 | GLOBAL |  | sceSifQueryBlockSize | sceSifQueryBlockSize |
| 0x0040c8c8 | 256 | LOCAL |  | _lf_bind | _lf_bind |
| 0x0040c9c8 | 140 | LOCAL |  | _lf_version | _lf_version |
| 0x0040ca58 | 56 | GLOBAL |  | sceSifLoadFileReset | sceSifLoadFileReset |
| 0x0040ca90 | 520 | LOCAL |  | _sceSifLoadModuleBuffer | _sceSifLoadModuleBuffer |
| 0x0040cc98 | 520 | GLOBAL |  | sceSifStopModule | sceSifStopModule |
| 0x0040cea0 | 144 | GLOBAL |  | sceSifUnloadModule | sceSifUnloadModule |
| 0x0040cf30 | 160 | GLOBAL |  | sceSifSearchModuleByName | sceSifSearchModuleByName |
| 0x0040cfd0 | 144 | GLOBAL |  | sceSifSearchModuleByAddress | sceSifSearchModuleByAddress |
| 0x0040d060 | 28 | GLOBAL |  | sceSifLoadModuleBuffer | sceSifLoadModuleBuffer |
| 0x0040d080 | 28 | GLOBAL |  | sceSifLoadStartModuleBuffer | sceSifLoadStartModuleBuffer |
| 0x0040d0a0 | 548 | LOCAL |  | _sceSifLoadModule | _sceSifLoadModule |
| 0x0040d2c8 | 32 | GLOBAL |  | sceSifLoadModule | sceSifLoadModule |
| 0x0040d2e8 | 28 | GLOBAL |  | sceSifLoadStartModule | sceSifLoadStartModule |
| 0x0040d308 | 260 | LOCAL |  | _sceSifLoadElfPart | _sceSifLoadElfPart |
| 0x0040d410 | 28 | GLOBAL |  | sceSifLoadElfPart | sceSifLoadElfPart |
| 0x0040d430 | 40 | GLOBAL |  | sceSifLoadElf | sceSifLoadElf |
| 0x0040d458 | 236 | GLOBAL |  | sceSifGetIopAddr | sceSifGetIopAddr |
| 0x0040d548 | 220 | GLOBAL |  | sceSifSetIopAddr | sceSifSetIopAddr |
| 0x0040d628 | 316 | GLOBAL |  | sceSifResetIop | sceSifResetIop |
| 0x0040d768 | 40 | GLOBAL |  | sceSifIsAliveIop | sceSifIsAliveIop |
| 0x0040d790 | 56 | GLOBAL |  | sceSifSyncIop | sceSifSyncIop |
| 0x0040d7c8 | 272 | GLOBAL |  | sceSifRebootIop | sceSifRebootIop |
| 0x0040d8d8 | 96 | GLOBAL |  | SetTLBHandler | SetTLBHandler |
| 0x0040d938 | 132 | GLOBAL |  | SetDebugHandler | SetDebugHandler |
| 0x0040d9c0 | 16 | LOCAL |  | Copy | Copy |
| 0x0040d9d0 | 56 | LOCAL |  | kCopy | kCopy |
| 0x0040da08 | 16 | LOCAL |  | GetEntryAddress | GetEntryAddress |
| 0x0040da18 | 16 | LOCAL |  | setup | setup |
| 0x0040da28 | 196 | GLOBAL |  | InitTLBFunctions | InitTLBFunctions |
| 0x0040daf0 | 16 | GLOBAL |  | PutTLBEntry | PutTLBEntry |
| 0x0040db00 | 16 | GLOBAL |  | iPutTLBEntry | iPutTLBEntry |
| 0x0040db10 | 16 | LOCAL |  | _SetTLBEntry | _SetTLBEntry |
| 0x0040db20 | 48 | GLOBAL |  | SetTLBEntry | SetTLBEntry |
| 0x0040db50 | 16 | GLOBAL |  | iSetTLBEntry | iSetTLBEntry |
| 0x0040db60 | 16 | GLOBAL |  | GetTLBEntry | GetTLBEntry |
| 0x0040db70 | 16 | GLOBAL |  | iGetTLBEntry | iGetTLBEntry |
| 0x0040db80 | 16 | GLOBAL |  | ProbeTLBEntry | ProbeTLBEntry |
| 0x0040db90 | 16 | GLOBAL |  | iProbeTLBEntry | iProbeTLBEntry |
| 0x0040dba0 | 16 | GLOBAL |  | ExpandScratchPad | ExpandScratchPad |
| 0x0040dbb0 | 64 | GLOBAL |  | InitTLB | InitTLB |
| 0x0040dbf0 | 500 | LOCAL |  | InitTLB32MB | InitTLB32MB |
| 0x0040de00 | 284 | GLOBAL |  | _kTLBException | _kTLBException |
| 0x0040df40 | 244 | GLOBAL |  | _kExitTLBHandler | _kExitTLBHandler |
| 0x0040e040 | 284 | GLOBAL |  | _kDebugException | _kDebugException |
| 0x0040e160 | 72 | GLOBAL |  | DIntr | DIntr |
| 0x0040e1a8 | 24 | GLOBAL |  | EIntr | EIntr |
| 0x0040e1c0 | 72 | LOCAL |  | supplement_crt0 | supplement_crt0 |
| 0x0040e208 | 16 | LOCAL |  | Copy | Copy |
| 0x0040e218 | 56 | LOCAL |  | kCopy | kCopy |
| 0x0040e250 | 64 | LOCAL |  | kFindAddress | kFindAddress |
| 0x0040e290 | 16 | LOCAL |  | FindAddress | FindAddress |
| 0x0040e2a0 | 52 | GLOBAL |  | GetSystemCallEntry | GetSystemCallEntry |
| 0x0040e2d8 | 252 | LOCAL |  | InitSystemCallTableAddress | InitSystemCallTableAddress |
| 0x0040e3d8 | 16 | LOCAL |  | setup | setup |
| 0x0040e3e8 | 60 | GLOBAL |  | _InitSys | _InitSys |
| 0x0040e428 | 16 | LOCAL |  | setup | setup |
| 0x0040e438 | 16 | LOCAL |  | Copy | Copy |
| 0x0040e448 | 56 | LOCAL |  | kCopy | kCopy |
| 0x0040e480 | 16 | LOCAL |  | GetEntryAddress | GetEntryAddress |
| 0x0040e490 | 104 | LOCAL |  | PatchIsNeeded | PatchIsNeeded |
| 0x0040e4f8 | 172 | GLOBAL |  | InitExecPS2 | InitExecPS2 |
| 0x0040e5a8 | 16 | LOCAL |  | Copy | Copy |
| 0x0040e5b8 | 48 | LOCAL |  | kCopy | kCopy |
| 0x0040e5e8 | 16 | LOCAL |  | setup | setup |
| 0x0040e5f8 | 280 | LOCAL |  | SetArg | SetArg |
| 0x0040e710 | 8 | LOCAL |  | TerminateLibrary | TerminateLibrary |
| 0x0040e718 | 112 | GLOBAL |  | ExecPS2 | ExecPS2 |
| 0x0040e788 | 72 | GLOBAL |  | LoadExecPS2 | LoadExecPS2 |
| 0x0040e7d0 | 40 | GLOBAL |  | Exit | Exit |
| 0x0040e7f8 | 76 | GLOBAL |  | ExecOSD | ExecOSD |
| 0x0040e848 | 16 | LOCAL |  | setup | setup |
| 0x0040e858 | 16 | LOCAL |  | Copy | Copy |
| 0x0040e868 | 56 | LOCAL |  | kCopy | kCopy |
| 0x0040e8a0 | 16 | LOCAL |  | GetEntryAddress | GetEntryAddress |
| 0x0040e8b0 | 212 | GLOBAL |  | InitAlarm | InitAlarm |
| 0x0040e990 | 28 | GLOBAL | I3D_animation_set | {ctor} | __ct__17I3D_animation_setFv |
| 0x0040e9b0 | 84 | GLOBAL | I3D_animation_set | {dtor} | __dt__17I3D_animation_setFv |
| 0x0040ea10 | 68 | GLOBAL | I3D_animation_set | Close | Close__17I3D_animation_setFv |
| 0x0040ea60 | 80 | GLOBAL | I3D_animation_set | Release | Release__17I3D_animation_setFv |
| 0x0040eab0 | 596 | GLOBAL | I3D_animation_set | Open | Open__17I3D_animation_setFPCcUi |
| 0x0040ed10 | 32 | GLOBAL | I3D_animation_set | NumAnimations | NumAnimations__17I3D_animation_setCFv |
| 0x0040ed30 | 44 | GLOBAL | I3D_animation_set | GetEndTime | GetEndTime__17I3D_animation_setCFv |
| 0x0040ed60 | 32 | GLOBAL | I3D_animation_set | GetAnims | GetAnims__17I3D_animation_setCFv |
| 0x0040ed80 | 144 | GLOBAL | I3D_animation_set | GetAnimIndex | GetAnimIndex__17I3D_animation_setCFPCc |
| 0x0040ee10 | 260 | GLOBAL | I3D_animation_set | GetPosKey | GetPosKey__17I3D_animation_setCFUiUiRiR8S_vector |
| 0x0040ef20 | 268 | GLOBAL | I3D_animation_set | NumNoteKeys | NumNoteKeys__17I3D_animation_setCFUiPi |
| 0x0040f030 | 320 | GLOBAL | I3D_animation_set | GetNoteKey | GetNoteKey__17I3D_animation_setCFUiUiRiRs |
| 0x0040f170 | 100 | GLOBAL | I3D_area | BogusConstructor | BogusConstructor__8I3D_areaFv |
| 0x0040f1e0 | 180 | GLOBAL | I3D_area | BogusDestructor | BogusDestructor__8I3D_areaFv |
| 0x0040f2a0 | 252 | GLOBAL | I3D_area | AddToRemoveFromSceneRing | AddToRemoveFromSceneRing__8I3D_areaFP9I3D_scene |
| 0x0040f3a0 | 208 | GLOBAL | I3D_area | LinkTo | LinkTo__8I3D_areaFP9I3D_frameUi |
| 0x0040f470 | 308 | GLOBAL | I3D_area | UpdateV | UpdateV__8I3D_areaFv |
| 0x0040f5b0 | 140 | GLOBAL | I3D_area | Duplicate | Duplicate__8I3D_areaFP9I3D_frame |
| 0x0040f640 | 488 | GLOBAL | I3D_area | Build | Build__8I3D_areaFPC8S_vectorUiPCUsUiUi |
| 0x0040f830 | 8 | 13 | S_plane | {ctor} | __ct__7S_planeFv |
| 0x0040f840 | 132 | GLOBAL | I3D_area | CheckPoint | CheckPoint__8I3D_areaFRC8S_vector |
| 0x0040f8d0 | 468 | GLOBAL | I3D_area | AddSound | AddSound__8I3D_areaFP9I3D_sound |
| 0x0040fab0 | 388 | GLOBAL | I3D_area | DeleteSound | DeleteSound__8I3D_areaFP9I3D_sound |
| 0x0040fc40 | 100 | GLOBAL | I3D_area | LoadImage | LoadImage__8I3D_areaFv |
| 0x0040fcb0 | 128 | 13 | I3D_area | {dtor} | __dt__8I3D_areaFv |
| 0x0040fd30 | 36 | 13 | I3D_area | SetSwitchFadeTime | SetSwitchFadeTime__8I3D_areaFUi |
| 0x0040fd60 | 8 | 13 | I3D_area | GetSwitchFadeTime | GetSwitchFadeTime__8I3D_areaFv |
| 0x0040fd70 | 48 | GLOBAL | I3D_billboard | BogusConstructor | BogusConstructor__13I3D_billboardFv |
| 0x0040fda0 | 8 | GLOBAL | I3D_billboard | SetSpace | SetSpace__13I3D_billboardFb |
| 0x0040fdb0 | 252 | GLOBAL | I3D_billboard | UpdateBoundProc | UpdateBoundProc__13I3D_billboardFv |
| 0x0040feb0 | 52 | GLOBAL | I3D_billboard | SetMesh | SetMesh__13I3D_billboardFP15I3D_mesh_objectb |
| 0x0040fef0 | 492 | GLOBAL | I3D_billboard | Duplicate | Duplicate__13I3D_billboardFP9I3D_frame |
| 0x004100e0 | 6216 | GLOBAL | I3D_billboard | ProductBillMatrix | ProductBillMatrix__13I3D_billboardFR8S_vectorP8S_matrix |
| 0x00411930 | 1376 | GLOBAL | I3D_billboard | m_AddFGroups | m_AddFGroups__13I3D_billboardFv |
| 0x00411e90 | 8 | 13 | I3D_material | UpdateEnvTexture | UpdateEnvTexture__12I3D_materialFv |
| 0x00411ea0 | 52 | GLOBAL | I3D_billboard | LoadImage | LoadImage__13I3D_billboardFv |
| 0x00411ee0 | 152 | GLOBAL | I3D_lensflare | BogusConstructor | BogusConstructor__13I3D_lensflareFv |
| 0x00411f80 | 116 | GLOBAL | I3D_lensflare | BogusDestructor | BogusDestructor__13I3D_lensflareFv |
| 0x00412000 | 8 | GLOBAL | I3D_lensflare | SetSpace | SetSpace__13I3D_lensflareFb |
| 0x00412010 | 608 | GLOBAL | I3D_lensflare | Duplicate | Duplicate__13I3D_lensflareFP9I3D_frame |
| 0x00412270 | 184 | 13 | C_list<11GLOW_STRUCT> | reserve | reserve__21C_list<11GLOW_STRUCT>Fi |
| 0x00412330 | 5644 | GLOBAL | I3D_lensflare | Draw | Draw__13I3D_lensflareFb |
| 0x00413940 | 476 | GLOBAL | I3D_lensflare | LoadImage | LoadImage__13I3D_lensflareFv |
| 0x00413b20 | 172 | 13 | I3D_lensflare | {dtor} | __dt__13I3D_lensflareFv |
| 0x00413bd0 | 8 | 13 | I3D_visual | m_AddFGroups | m_AddFGroups__10I3D_visualFv |
| 0x00413be0 | 8 | 13 | I3D_visual | m_RenderFGroup | m_RenderFGroup__10I3D_visualFP21RENDER_FG_HELP_STRUCT |
| 0x00413bf0 | 164 | 13 | I3D_billboard | {dtor} | __dt__13I3D_billboardFv |
| 0x00413ca0 | 396 | GLOBAL | I3D_camera | BogusConstructor | BogusConstructor__10I3D_cameraFv |
| 0x00413e30 | 24 | GLOBAL | I3D_camera | BogusDestructor | BogusDestructor__10I3D_cameraFv |
| 0x00413e50 | 464 | GLOBAL | I3D_camera | Duplicate | Duplicate__10I3D_cameraFP9I3D_frame |
| 0x00414020 | 996 | GLOBAL | I3D_camera | UpdateProjMatrix | UpdateProjMatrix__10I3D_cameraFv |
| 0x00414410 | 120 | GLOBAL | I3D_camera | SetFOV | SetFOV__10I3D_cameraFf |
| 0x00414490 | 120 | GLOBAL | I3D_camera | SetNearFOV | SetNearFOV__10I3D_cameraFf |
| 0x00414510 | 124 | GLOBAL | I3D_camera | SetRange | SetRange__10I3D_cameraFff |
| 0x00414590 | 124 | GLOBAL | I3D_camera | SetNearRange | SetNearRange__10I3D_cameraFff |
| 0x00414610 | 64 | GLOBAL | I3D_camera | SetOrthoScale | SetOrthoScale__10I3D_cameraFf |
| 0x00414650 | 8 | GLOBAL | I3D_camera | SetOrthogonal | SetOrthogonal__10I3D_cameraFb |
| 0x00414660 | 100 | GLOBAL | I3D_camera | SetAspectRatio | SetAspectRatio__10I3D_cameraFf |
| 0x004146d0 | 124 | GLOBAL | I3D_camera | UpdateSector | UpdateSector__10I3D_cameraFv |
| 0x00414750 | 3308 | GLOBAL | I3D_camera | UpdateV | UpdateV__10I3D_cameraFv |
| 0x00415440 | 96 | 13 | I3D_camera | {dtor} | __dt__10I3D_cameraFv |
| 0x004154a0 | 4272 | LOCAL |  | testSinglemesh__FP14I3D_singlemeshRC8S_vectorRC8S_vectorR8S_vector | testSinglemesh__FP14I3D_singlemeshRC8S_vectorRC8S_vectorR8S_vector |
| 0x00416550 | 11524 | LOCAL |  | testVisual__FP10I3D_visual | testVisual__FP10I3D_visual |
| 0x00419260 | 1768 | LOCAL |  | testSubtree__FP9I3D_frame | testSubtree__FP9I3D_frame |
| 0x00419950 | 536 | LOCAL |  | DefaultTexelCallback__FRC8S_vectorfP9I3D_framePvPv | DefaultTexelCallback__FRC8S_vectorfP9I3D_framePvPv |
| 0x00419b70 | 1256 | GLOBAL | I3D_scene | TestColGeo | TestColGeo__9I3D_sceneCFRC8S_vectorRC8S_vectorUiPfP8S_vectorPFRC8S_vectorfP9I3D_framePv_12COLL_ENUMRETPvPFRC8S_vectorfP9I3D_framePvPv_iP9I3D_framei |
| 0x0041a060 | 1264 | GLOBAL | I3D_scene | TestColGeo1 | TestColGeo1__9I3D_sceneCFRC8S_vectorRC8S_vectorUiP8S_vectorP8S_vectorPFRC8S_vectorfP9I3D_framePv_12COLL_ENUMRETPvPFRC8S_vectorfP9I3D_framePvPv_ii |
| 0x0041a550 | 1144 | GLOBAL | I3D_scene | TestColHierarchy | TestColHierarchy__9I3D_sceneCFRC8S_vectorRC8S_vectorP9I3D_frameR13I3D_COLLISIONUiP9I3D_frameiPFRC8S_vectorfP9I3D_framePv_12COLL_ENUMRET |
| 0x0041a9d0 | 10436 | LOCAL |  | sphTestVisual__FP10I3D_visual | sphTestVisual__FP10I3D_visual |
| 0x0041d2a0 | 1380 | LOCAL |  | sphTestSubtree__FP9I3D_frame | sphTestSubtree__FP9I3D_frame |
| 0x0041d810 | 948 | GLOBAL | I3D_scene | TestColSphere | TestColSphere__9I3D_sceneCFRC8S_vectorRC8S_vectorfUiP8S_vectorP9I3D_framePFRC8S_vectorfP9I3D_framePv_12COLL_ENUMRET |
| 0x0041dbd0 | 996 | GLOBAL | I3D_scene | TestColSphereHierarchy | TestColSphereHierarchy__9I3D_sceneCFRC8S_vectorRC8S_vectorfP9I3D_frameUiP8S_vectorP9I3D_framePFRC8S_vectorfP9I3D_framePv_12COLL_ENUMRET |
| 0x0041dfc0 | 1768 | GLOBAL | I3D_scene | TestColSphereVisual | TestColSphereVisual__9I3D_sceneCFRC8S_vectorRC8S_vectorfP10I3D_visualUiP8S_vector |
| 0x0041e6b0 | 600 | GLOBAL | I3D_scene | SphereToSphereD | SphereToSphereD__9I3D_sceneCFfRC8S_vectorRC8S_vectorfRC8S_vectorRC8S_vector |
| 0x0041e910 | 92 | GLOBAL | I3D_driver | {ctor} | __ct__10I3D_driverFv |
| 0x0041e970 | 92 | 13 | C_list<P8ITexture> | {dtor} | __dt__18C_list<P8ITexture>Fv |
| 0x0041e9d0 | 92 | 13 | C_list<P10I3D_sector> | {dtor} | __dt__21C_list<P10I3D_sector>Fv |
| 0x0041ea30 | 92 | 13 | C_list<P9I3D_sound> | {dtor} | __dt__19C_list<P9I3D_sound>Fv |
| 0x0041ea90 | 244 | GLOBAL | I3D_driver | {dtor} | __dt__10I3D_driverFv |
| 0x0041eb90 | 16 | GLOBAL | I3D_driver | Tick | Tick__10I3D_driverFi |
| 0x0041eba0 | 8 | GLOBAL | I3D_driver | GetTickTime | GetTickTime__10I3D_driverCFv |
| 0x0041ebb0 | 872 | GLOBAL | I3D_driver | Init | Init__10I3D_driverFUi |
| 0x0041ef20 | 340 | GLOBAL | I3D_driver | DeleteReleasedObjects | DeleteReleasedObjects__10I3D_driverFv |
| 0x0041f080 | 208 | GLOBAL | I3D_driver | Close | Close__10I3D_driverFv |
| 0x0041f150 | 192 | GLOBAL | I3D_driver | GetStats | GetStats__10I3D_driverF13I3DSTATSINDEXPv |
| 0x0041f210 | 1428 | GLOBAL | I3D_driver | SetState | SetState__10I3D_driverF15I3D_RENDERSTATEi |
| 0x0041f7b0 | 48 | GLOBAL | I3D_driver | GetState | GetState__10I3D_driverCF15I3D_RENDERSTATE |
| 0x0041f7e0 | 1012 | GLOBAL | I3D_driver | DrawPoint | DrawPoint__10I3D_driverFRC8S_vectorRC8S_vectorUi |
| 0x0041fbe0 | 1032 | GLOBAL | I3D_driver | DrawPoint2D | DrawPoint2D__10I3D_driverFiiRC8S_vectorUi |
| 0x0041fff0 | 1052 | GLOBAL | I3D_driver | DrawLine | DrawLine__10I3D_driverFRC8S_vectorRC8S_vectorRC8S_vectorUi |
| 0x00420410 | 1332 | GLOBAL | I3D_driver | DrawLines | DrawLines__10I3D_driverFPC8S_vectorUiPCUsUiRC8S_vectorUiUi |
| 0x00420950 | 2508 | GLOBAL | I3D_driver | DrawTriangles | DrawTriangles__10I3D_driverFPCvUiPCUsUiRC8S_vectorUiUiP13I3D_text_coor |
| 0x00421320 | 1524 | GLOBAL | I3D_driver | DrawCircle2D | DrawCircle2D__10I3D_driverFRC9S_vector2fRC8S_vectoriUi |
| 0x00421920 | 2064 | GLOBAL | I3D_driver | DrawText | DrawText__10I3D_driverFRC8S_vectorPCcUcf |
| 0x00422130 | 2212 | GLOBAL | I3D_driver | DrawText2D | DrawText2D__10I3D_driverFiiPCcUcf |
| 0x004229e0 | 1656 | GLOBAL | I3D_driver | DrawSprite | DrawSprite__10I3D_driverFRC8S_vectorP12I3D_materialUcf |
| 0x00423060 | 352 | GLOBAL | I3D_driver | DrawBox | DrawBox__10I3D_driverFRC8I3D_bboxPC8S_vectorPC8S_vectorUs |
| 0x004231c0 | 324 | GLOBAL | I3D_driver | DrawSphere | DrawSphere__10I3D_driverFRC8S_matrixRC11I3D_bsphereRC8S_vectorUs |
| 0x00423310 | 96 | GLOBAL | I3D_driver | CreateMaterial | CreateMaterial__10I3D_driverFv |
| 0x00423370 | 96 | GLOBAL | I3D_driver | CreateAnimMaterial | CreateAnimMaterial__10I3D_driverFv |
| 0x004233d0 | 16 | GLOBAL | I3D_driver | CreateTrifaceArray | CreateTrifaceArray__10I3D_driverFi |
| 0x004233e0 | 44 | GLOBAL | I3D_driver | CreateFaceGroup | CreateFaceGroup__10I3D_driverFv |
| 0x00423410 | 44 | GLOBAL | I3D_driver | CreateMeshLevel | CreateMeshLevel__10I3D_driverFv |
| 0x00423440 | 44 | GLOBAL | I3D_driver | CreateMesh | CreateMesh__10I3D_driverFv |
| 0x00423470 | 44 | GLOBAL | I3D_driver | CreateAnimationSet | CreateAnimationSet__10I3D_driverFv |
| 0x004234a0 | 44 | GLOBAL | I3D_driver | CreatePose | CreatePose__10I3D_driverFv |
| 0x004234d0 | 44 | GLOBAL | I3D_driver | CreatePoseSet | CreatePoseSet__10I3D_driverFv |
| 0x00423500 | 1584 | GLOBAL | I3D_driver | GetFreeFrame | GetFreeFrame__10I3D_driverF14I3D_FRAME_TYPE |
| 0x00423b30 | 92 | 13 | C_list<P9I3D_light> | {dtor} | __dt__19C_list<P9I3D_light>Fv |
| 0x00423b90 | 92 | 13 | C_list<P8I3D_area> | {dtor} | __dt__18C_list<P8I3D_area>Fv |
| 0x00423bf0 | 1804 | GLOBAL | I3D_driver | GetFreeVisual | GetFreeVisual__10I3D_driverFi |
| 0x00424300 | 164 | 13 | I3D_singlemesh | {dtor} | __dt__14I3D_singlemeshFv |
| 0x004243b0 | 56 | 13 | I3D_visual | {ctor} | __ct__10I3D_visualFv |
| 0x004243f0 | 136 | 13 | I3D_object | {dtor} | __dt__10I3D_objectFv |
| 0x00424480 | 20 | 13 | I3D_frame | {ctor} | __ct__9I3D_frameFv |
| 0x004244a0 | 108 | 13 | I3D_visual | {dtor} | __dt__10I3D_visualFv |
| 0x00424510 | 8 | 13 | I3D_bsphere | {ctor} | __ct__11I3D_bsphereFv |
| 0x00424520 | 140 | GLOBAL |  | FreeFrame__FP9I3D_frame | FreeFrame__FP9I3D_frame |
| 0x004245b0 | 8 | GLOBAL | I3D_driver | CreateFrame | CreateFrame__10I3D_driverF14I3D_FRAME_TYPE |
| 0x004245c0 | 64 | GLOBAL | I3D_driver | CreateVisual | CreateVisual__10I3D_driverF15I3D_VISUAL_TYPE |
| 0x00424600 | 292 | GLOBAL | I3D_driver | EnumVisualTypes | EnumVisualTypes__10I3D_driverCFPFUiPCc_vb |
| 0x00424730 | 280 | GLOBAL |  | FrameType2StringFunc__F14I3D_FRAME_TYPE | FrameType2StringFunc__F14I3D_FRAME_TYPE |
| 0x00424850 | 8 | GLOBAL | I3D_driver | FrameType2String | FrameType2String__10I3D_driverCF14I3D_FRAME_TYPE |
| 0x00424860 | 356 | GLOBAL | I3D_driver | VisualType2String | VisualType2String__10I3D_driverCF15I3D_VISUAL_TYPE |
| 0x004249d0 | 328 | GLOBAL | I3D_driver | FindFrameFast | FindFrameFast__10I3D_driverCFP9I3D_scenePCc |
| 0x00424b20 | 292 | GLOBAL |  | FindFrameFast___10I3D_driverFPCcP9I3D_frame | FindFrameFast___10I3D_driverFPCcP9I3D_frame |
| 0x00424c50 | 156 | GLOBAL | I3D_driver | OptimalizeFastFindData | OptimalizeFastFindData__10I3D_driverFv |
| 0x00424cf0 | 560 | GLOBAL |  | OFFD_RebuildBranchOfFFTree___10I3D_driverFPP9I3D_frameii | OFFD_RebuildBranchOfFFTree___10I3D_driverFPP9I3D_frameii |
| 0x00424f20 | 180 | GLOBAL | I3D_driver | EnumFFTree | EnumFFTree__10I3D_driverCFPFP9I3D_frameUi_10I3DENUMRETUi |
| 0x00424fe0 | 332 | 13 | I3D_driver::_hlp$3643I3d_driver_cpp | enumSubtree | enumSubtree__Q210I3D_driver23_hlp$3643I3d_driver_cppFPFP9I3D_frameUi_10I3DENUMRETUiP9I3D_frame |
| 0x00425130 | 112 | GLOBAL | I3D_driver | DeleteFromSceneTickList | DeleteFromSceneTickList__10I3D_driverFP9I3D_frame |
| 0x004251a0 | 8 | GLOBAL | I3D_driver | DeleteFromAllShadows | DeleteFromAllShadows__10I3D_driverFP9I3D_frame |
| 0x004251b0 | 8 | GLOBAL | I3D_driver | DeleteFromAllTargets | DeleteFromAllTargets__10I3D_driverFP9I3D_frame |
| 0x004251c0 | 8 | GLOBAL | I3D_driver | m_RecomputeUsedVRAM | m_RecomputeUsedVRAM__10I3D_driverFv |
| 0x004251d0 | 120 | LOCAL |  | SetLightnessHR__FP9I3D_frame | SetLightnessHR__FP9I3D_frame |
| 0x00425250 | 120 | LOCAL |  | SetTransparencyHR__FP9I3D_frame | SetTransparencyHR__FP9I3D_frame |
| 0x004252d0 | 12 | GLOBAL | I3D_driver | SetLightness | SetLightness__10I3D_driverFP9I3D_framef |
| 0x004252e0 | 12 | GLOBAL | I3D_driver | SetTransparency | SetTransparency__10I3D_driverFP9I3D_framef |
| 0x004252f0 | 544 | LOCAL |  | GetHRBboxHR__FP9I3D_frame | GetHRBboxHR__FP9I3D_frame |
| 0x00425510 | 84 | GLOBAL | I3D_driver | GetHRBbox | GetHRBbox__10I3D_driverFP9I3D_frame |
| 0x00425570 | 520 | LOCAL |  | GetHRBboxExHR__FP9I3D_frameRC8S_matrix | GetHRBboxExHR__FP9I3D_frameRC8S_matrix |
| 0x00425780 | 240 | GLOBAL | I3D_driver | GetHRBboxEx | GetHRBboxEx__10I3D_driverFP9I3D_frame |
| 0x00425870 | 92 | 13 | C_list<7LVERTEX> | {dtor} | __dt__16C_list<7LVERTEX>Fv |
| 0x004258d0 | 92 | 13 | C_list<8TLVERTEX> | {dtor} | __dt__17C_list<8TLVERTEX>Fv |
| 0x00425930 | 92 | 13 | C_list<P12I3D_material> | {dtor} | __dt__23C_list<P12I3D_material>Fv |
| 0x00425990 | 64 | GLOBAL | I3D_dummy | BogusConstructor | BogusConstructor__9I3D_dummyFv |
| 0x004259d0 | 124 | GLOBAL | I3D_dummy | BogusDestructor | BogusDestructor__9I3D_dummyFv |
| 0x00425a50 | 76 | GLOBAL | I3D_dummy | Duplicate | Duplicate__9I3D_dummyFP9I3D_frame |
| 0x00425aa0 | 764 | GLOBAL | I3D_dummy | Normalize | Normalize__9I3D_dummyFUi |
| 0x00425da0 | 52 | GLOBAL | I3D_dummy | LoadImage | LoadImage__9I3D_dummyFv |
| 0x00425de0 | 96 | 13 | I3D_dummy | {dtor} | __dt__9I3D_dummyFv |
| 0x00425e40 | 76 | GLOBAL | I3D_frame | {dtor} | __dt__9I3D_frameFv |
| 0x00425e90 | 200 | GLOBAL | I3D_frame | BogusConstructor | BogusConstructor__9I3D_frameFv |
| 0x00425f60 | 260 | GLOBAL | I3D_frame | BogusDestructor | BogusDestructor__9I3D_frameFv |
| 0x00426070 | 60 | GLOBAL | I3D_frame | GetVisualType | GetVisualType__9I3D_frameCFv |
| 0x004260b0 | 136 | GLOBAL | I3D_frame | Release | Release__9I3D_frameFv |
| 0x00426140 | 108 | GLOBAL | I3D_frame | DeleteFromOwner | DeleteFromOwner__9I3D_frameFv |
| 0x004261b0 | 1500 | GLOBAL | I3D_frame | SetDir | SetDir__9I3D_frameFRC8S_vectorf |
| 0x00426790 | 352 | GLOBAL | I3D_frame | SetWorldDir | SetWorldDir__9I3D_frameFRC8S_vectorf |
| 0x004268f0 | 392 | GLOBAL | I3D_frame | SetWorldPos | SetWorldPos__9I3D_frameFRC8S_vector |
| 0x00426a80 | 504 | GLOBAL | I3D_frame | SetWorldPosDir | SetWorldPosDir__9I3D_frameFRC8S_vectorRC8S_vectorf |
| 0x00426c80 | 532 | GLOBAL | I3D_frame | SetWorldPosDir | SetWorldPosDir__9I3D_frameFRC8S_vectorRC8S_vectorRC8S_vector |
| 0x00426ea0 | 432 | GLOBAL | I3D_frame | SetWorldPosRot | SetWorldPosRot__9I3D_frameFRC8S_vectorRC6S_quat |
| 0x00427050 | 60 | GLOBAL | I3D_frame | SetCallback | SetCallback__9I3D_frameFPFP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi_UiUi |
| 0x00427090 | 48 | GLOBAL | I3D_frame | SetOn | SetOn__9I3D_frameFb |
| 0x004270c0 | 216 | GLOBAL | I3D_frame | RemoveFromFFTree | RemoveFromFFTree__9I3D_frameFv |
| 0x004271a0 | 304 | GLOBAL | I3D_frame | AddToFFTree | AddToFFTree__9I3D_frameFv |
| 0x004272d0 | 84 | GLOBAL | I3D_frame | OFFD_CountFrames | OFFD_CountFrames__9I3D_frameFv |
| 0x00427330 | 84 | GLOBAL |  | OFFD_CountFrames___9I3D_frameFv | OFFD_CountFrames___9I3D_frameFv |
| 0x00427390 | 104 | GLOBAL |  | AlphabeticalListOfFrames___9I3D_frameFv | AlphabeticalListOfFrames___9I3D_frameFv |
| 0x00427400 | 336 | GLOBAL | I3D_frame | SetName | SetName__9I3D_frameFPCc |
| 0x00427550 | 384 | GLOBAL | I3D_frame | SetModelName | SetModelName__9I3D_frameFPCc |
| 0x004276d0 | 188 | GLOBAL | I3D_frame | ResetModelName | ResetModelName__9I3D_frameFv |
| 0x00427790 | 1412 | GLOBAL | I3D_frame | LinkTo | LinkTo__9I3D_frameFP9I3D_frameUi |
| 0x00427d20 | 32 | 13 | I3D_frame::EnumHelp$1368I3d_frame_cpp | I3D_EnumProc | I3D_EnumProc__Q29I3D_frame26EnumHelp$1368I3d_frame_cppFP9I3D_frameUi |
| 0x00427d40 | 32 | 13 | I3D_frame::EnumHelp$1365I3d_frame_cpp | I3D_EnumProc | I3D_EnumProc__Q29I3D_frame26EnumHelp$1365I3d_frame_cppFP9I3D_frameUi |
| 0x00427d60 | 32 | 13 | I3D_frame::EnumHelp$1362I3d_frame_cpp | I3D_EnumProc | I3D_EnumProc__Q29I3D_frame26EnumHelp$1362I3d_frame_cppFP9I3D_frameUi |
| 0x00427d80 | 36 | GLOBAL | I3D_frame | AddChild | AddChild__9I3D_frameFP9I3D_frame |
| 0x00427db0 | 112 | GLOBAL | I3D_frame | RemoveChild | RemoveChild__9I3D_frameFP9I3D_frame |
| 0x00427e20 | 52 | GLOBAL | I3D_frame | NumChildren | NumChildren__9I3D_frameCFv |
| 0x00427e60 | 52 | GLOBAL | I3D_frame | GetChild | GetChild__9I3D_frameCFi |
| 0x00427ea0 | 36 | GLOBAL |  | I3Dtoupper__Fi | I3Dtoupper__Fi |
| 0x00427ed0 | 856 | GLOBAL | I3D_frame | EnumSubtree | EnumSubtree__9I3D_frameCFPFP9I3D_frameUi_10I3DENUMRETUiUiPCc |
| 0x00428230 | 160 | GLOBAL | I3D_frame | EnumFrames | EnumFrames__9I3D_frameCFPFP9I3D_frameUi_10I3DENUMRETUiUiPCc |
| 0x004282d0 | 12 | GLOBAL |  | FindChildCallBack__FP9I3D_frameUi | FindChildCallBack__FP9I3D_frameUi |
| 0x004282e0 | 60 | GLOBAL | I3D_frame | FindChildFrame | FindChildFrame__9I3D_frameCFPCcUi |
| 0x00428320 | 796 | GLOBAL | I3D_frame | GetChildFrame | GetChildFrame__9I3D_frameFPCcUi |
| 0x00428640 | 372 | GLOBAL | I3D_frame | Duplicate | Duplicate__9I3D_frameFP9I3D_frame |
| 0x004287c0 | 224 | GLOBAL | I3D_frame | SetProperty | SetProperty__9I3D_frameFPCc |
| 0x004288a0 | 636 | GLOBAL | I3D_frame | UpdateWMatrixProc | UpdateWMatrixProc__9I3D_frameFv |
| 0x00428b20 | 264 | GLOBAL | I3D_frame | UpdateChildren | UpdateChildren__9I3D_frameFv |
| 0x00428c30 | 272 | GLOBAL | I3D_frame | Update | Update__9I3D_frameFv |
| 0x00428d40 | 76 | GLOBAL | I3D_frame | DiscardHRBoundVol | DiscardHRBoundVol__9I3D_frameFv |
| 0x00428d90 | 8 | GLOBAL | I3D_frame | UpdateV | UpdateV__9I3D_frameFv |
| 0x00428da0 | 1068 | GLOBAL | I3D_frame | UpdateHRBoundVolW | UpdateHRBoundVolW__9I3D_frameFv |
| 0x004291d0 | 32 | 13 | I3D_bbox | Absorb | Absorb__8I3D_bboxFRC8S_vector |
| 0x004291f0 | 132 | GLOBAL | I3D_frame | SetHRLightness | SetHRLightness__9I3D_frameFf |
| 0x00429280 | 132 | GLOBAL | I3D_frame | SetHRTransparency | SetHRTransparency__9I3D_frameFf |
| 0x00429310 | 8 | GLOBAL | I3D_frame | SaveImage | SaveImage__9I3D_frameFv |
| 0x00429320 | 372 | GLOBAL | I3D_frame | LoadImage | LoadImage__9I3D_frameFv |
| 0x004294a0 | 92 | 13 | C_list<P10I3D_target> | {dtor} | __dt__21C_list<P10I3D_target>Fv |
| 0x00429500 | 72 | GLOBAL | I3D_interpolator | {ctor} | __ct__16I3D_interpolatorFP11I3D_IPolSet |
| 0x00429550 | 96 | GLOBAL | I3D_interpolator | {dtor} | __dt__16I3D_interpolatorFv |
| 0x004295b0 | 8 | GLOBAL | I3D_interpolator | SetPoseBlend | SetPoseBlend__16I3D_interpolatorFf |
| 0x004295c0 | 4328 | GLOBAL | I3D_interpolator | Update | Update__16I3D_interpolatorFv |
| 0x0042a6b0 | 120 | GLOBAL | I3D_interpolator | SetPose | SetPose__16I3D_interpolatorFP8I3D_pose |
| 0x0042a730 | 92 | GLOBAL | I3D_IPolSet | {ctor} | __ct__11I3D_IPolSetFP9I3D_frame |
| 0x0042a790 | 116 | GLOBAL | I3D_IPolSet | {dtor} | __dt__11I3D_IPolSetFv |
| 0x0042a810 | 172 | GLOBAL | I3D_IPolSet | Clear | Clear__11I3D_IPolSetFv |
| 0x0042a8c0 | 484 | GLOBAL | I3D_IPolSet | Tick | Tick__11I3D_IPolSetFPCi |
| 0x0042aab0 | 464 | GLOBAL | I3D_IPolSet | Tick | Tick__11I3D_IPolSetFi |
| 0x0042ac80 | 772 | GLOBAL | I3D_IPolSet | SetAnimation | SetAnimation__11I3D_IPolSetFP17I3D_animation_setUif15I3D_ANIMOPTIONS |
| 0x0042af90 | 88 | GLOBAL | I3D_IPolSet | SetPose | SetPose__11I3D_IPolSetFP9I3D_frameP8I3D_pose |
| 0x0042aff0 | 88 | GLOBAL | I3D_IPolSet | SetPose | SetPose__11I3D_IPolSetFPCcP8I3D_pose |
| 0x0042b050 | 164 | GLOBAL | I3D_IPolSet | SetPose | SetPose__11I3D_IPolSetFP12I3D_pose_set |
| 0x0042b100 | 112 | GLOBAL | I3D_IPolSet | SetPoseBlend | SetPoseBlend__11I3D_IPolSetFf |
| 0x0042b170 | 172 | GLOBAL | I3D_IPolSet | SetPoseBlend | SetPoseBlend__11I3D_IPolSetFfP12I3D_pose_set |
| 0x0042b220 | 64 | GLOBAL | I3D_IPolSet | GetPoseBlend | GetPoseBlend__11I3D_IPolSetCFv |
| 0x0042b260 | 108 | GLOBAL | I3D_IPolSet | ResetAllPoses | ResetAllPoses__11I3D_IPolSetFv |
| 0x0042b2d0 | 232 | GLOBAL | I3D_IPolSet | m_releaseUnusedInterpolators | m_releaseUnusedInterpolators__11I3D_IPolSetFv |
| 0x0042b3c0 | 344 | GLOBAL | I3D_IPolSet | GetFrameInterpolator | GetFrameInterpolator__11I3D_IPolSetFP9I3D_frame |
| 0x0042b520 | 428 | GLOBAL | I3D_IPolSet | GetFrameInterpolator | GetFrameInterpolator__11I3D_IPolSetFPCc |
| 0x0042b6d0 | 48 | GLOBAL | I3D_joint | BogusConstructor | BogusConstructor__9I3D_jointFv |
| 0x0042b700 | 72 | GLOBAL | I3D_joint | BogusDestructor | BogusDestructor__9I3D_jointFv |
| 0x0042b750 | 168 | GLOBAL | I3D_joint | UpdateV | UpdateV__9I3D_jointFv |
| 0x0042b800 | 72 | GLOBAL | I3D_joint | Duplicate | Duplicate__9I3D_jointFP9I3D_frame |
| 0x0042b850 | 52 | GLOBAL | I3D_joint | LoadImage | LoadImage__9I3D_jointFv |
| 0x0042b890 | 96 | 13 | I3D_joint | {dtor} | __dt__9I3D_jointFv |
| 0x0042b8f0 | 1128 | GLOBAL | I3D_light | BogusConstructor | BogusConstructor__9I3D_lightFv |
| 0x0042bd60 | 188 | GLOBAL | I3D_light | BogusDestructor | BogusDestructor__9I3D_lightFv |
| 0x0042be20 | 684 | GLOBAL | I3D_light | SetLightType | SetLightType__9I3D_lightF13I3D_LIGHTTYPE |
| 0x0042c0d0 | 244 | GLOBAL | I3D_light | SetDiffuse | SetDiffuse__9I3D_lightFRC8S_vector |
| 0x0042c1d0 | 248 | GLOBAL | I3D_light | SetDiffuse | SetDiffuse__9I3D_lightFfff |
| 0x0042c2d0 | 244 | GLOBAL | I3D_light | SetSpecular | SetSpecular__9I3D_lightFRC8S_vector |
| 0x0042c3d0 | 244 | GLOBAL | I3D_light | SetAmbient | SetAmbient__9I3D_lightFRC8S_vector |
| 0x0042c4d0 | 240 | GLOBAL | I3D_light | SetDiffusePower | SetDiffusePower__9I3D_lightFf |
| 0x0042c5c0 | 240 | GLOBAL | I3D_light | SetSpecularPower | SetSpecularPower__9I3D_lightFf |
| 0x0042c6b0 | 240 | GLOBAL | I3D_light | SetAmbientPower | SetAmbientPower__9I3D_lightFf |
| 0x0042c7a0 | 180 | GLOBAL | I3D_light | SetRange | SetRange__9I3D_lightFf |
| 0x0042c860 | 128 | GLOBAL | I3D_light | SetAttenuations | SetAttenuations__9I3D_lightFfff |
| 0x0042c8e0 | 128 | GLOBAL | I3D_light | SetFogRange | SetFogRange__9I3D_lightFff |
| 0x0042c960 | 336 | GLOBAL | I3D_light | SetCone | SetCone__9I3D_lightFff |
| 0x0042cab0 | 900 | GLOBAL | I3D_light | UpdateV | UpdateV__9I3D_lightFv |
| 0x0042ce40 | 224 | GLOBAL | I3D_light | SetOn | SetOn__9I3D_lightFb |
| 0x0042cf20 | 152 | GLOBAL | I3D_light | SetMode | SetMode__9I3D_lightFUi |
| 0x0042cfc0 | 272 | GLOBAL | I3D_light | Duplicate | Duplicate__9I3D_lightFP9I3D_frame |
| 0x0042d0d0 | 640 | GLOBAL | I3D_light | LoadImage | LoadImage__9I3D_lightFv |
| 0x0042d350 | 128 | 13 | I3D_light | {dtor} | __dt__9I3D_lightFv |
| 0x0042d3d0 | 132 | GLOBAL | I3D_lit_object | BogusConstructor | BogusConstructor__14I3D_lit_objectFv |
| 0x0042d460 | 44 | GLOBAL | I3D_lit_object | BogusDestructor | BogusDestructor__14I3D_lit_objectFv |
| 0x0042d490 | 156 | 13 | I3D_lit_object | deletePrecomputedLitLODs | deletePrecomputedLitLODs__14I3D_lit_objectFv |
| 0x0042d530 | 80 | GLOBAL | I3D_lit_object | SetActiveLevel | SetActiveLevel__14I3D_lit_objectFi |
| 0x0042d580 | 64 | GLOBAL | I3D_lit_object | GetNumRectInfos | GetNumRectInfos__14I3D_lit_objectFii |
| 0x0042d5c0 | 64 | GLOBAL | I3D_lit_object | GetRectInfos | GetRectInfos__14I3D_lit_objectFii |
| 0x0042d600 | 944 | GLOBAL | I3D_lit_object | UpdateLMFromRectInfos | UpdateLMFromRectInfos__14I3D_lit_objectFii |
| 0x0042d9b0 | 108 | GLOBAL | I3D_lit_object | GetRectInfoFlags | GetRectInfoFlags__14I3D_lit_objectFiii |
| 0x0042da20 | 22132 | GLOBAL | I3D_lit_object | Construct | Construct__14I3D_lit_objectFP9I3D_sceneR8S_vectorfUiPF21LS3D_CALLBACK_MESSAGEUiUiPv_vPv |
| 0x004330a0 | 8 | 13 | FACE_STRUCT | {ctor} | __ct__11FACE_STRUCTFv |
| 0x004330b0 | 8 | 13 | VERTEX_STRUCT | {ctor} | __ct__13VERTEX_STRUCTFv |
| 0x004330c0 | 92 | 13 | C_list<15NEW_FACE_STRUCT> | {dtor} | __dt__25C_list<15NEW_FACE_STRUCT>Fv |
| 0x00433120 | 28 | 13 | C_list<15NEW_FACE_STRUCT> | {ctor} | __ct__25C_list<15NEW_FACE_STRUCT>Fv |
| 0x00433140 | 148 | GLOBAL | I3D_lit_object | Destruct | Destruct__14I3D_lit_objectFUi |
| 0x004331e0 | 2636 | GLOBAL | I3D_lit_object | Load | Load__14I3D_lit_objectFi |
| 0x00433c30 | 40 | GLOBAL | I3D_lit_object | Save | Save__14I3D_lit_objectFi |
| 0x00433c60 | 92 | GLOBAL | I3D_lit_object | SetMode | SetMode__14I3D_lit_objectFUi |
| 0x00433cc0 | 572 | GLOBAL | I3D_lit_object | Duplicate | Duplicate__14I3D_lit_objectFP9I3D_frame |
| 0x00433f00 | 340 | GLOBAL | I3D_lit_object | SetMesh | SetMesh__14I3D_lit_objectFP15I3D_mesh_objectb |
| 0x00434060 | 104 | GLOBAL | I3D_lit_object | UpdateVertices | UpdateVertices__14I3D_lit_objectFv |
| 0x004340d0 | 72 | GLOBAL | I3D_lit_object | UpdateVertices | UpdateVertices__14I3D_lit_objectFi |
| 0x00434120 | 124 | GLOBAL | I3D_lit_object | GetTotalLMMemory | GetTotalLMMemory__14I3D_lit_objectFv |
| 0x004341a0 | 1620 | GLOBAL | I3D_lit_object | m_AddFGroups | m_AddFGroups__14I3D_lit_objectFv |
| 0x00434800 | 3120 | GLOBAL | I3D_lit_object | m_RenderFGroup | m_RenderFGroup__14I3D_lit_objectFP21RENDER_FG_HELP_STRUCT |
| 0x00435430 | 200 | GLOBAL | litLod | {ctor} | __ct__6litLodFv |
| 0x00435500 | 420 | GLOBAL | litLod | {dtor} | __dt__6litLodFv |
| 0x004356b0 | 1900 | GLOBAL | litLod | Create | Create__6litLodFP14I3D_mesh_levelP15I3D_mesh_objectUi |
| 0x00435e20 | 112 | GLOBAL | litLod | SetNumFGI | SetNumFGI__6litLodFi |
| 0x00435e90 | 40 | GLOBAL | litLod | AddFGindex | AddFGindex__6litLodFPUsi |
| 0x00435ec0 | 84 | GLOBAL | litLod | m_GetLVertex | m_GetLVertex__6litLodFRUsRUs |
| 0x00435f20 | 1084 | GLOBAL | litLod | postProcess | postProcess__6litLodFP14I3D_mesh_levelbP14I3D_lit_object |
| 0x00436360 | 164 | 13 | I3D_lit_object | {dtor} | __dt__14I3D_lit_objectFv |
| 0x00436410 | 20 | 13 |  | T_swap<Us>__FRUsRUs | T_swap<Us>__FRUsRUs |
| 0x00436430 | 20 | 13 |  | T_swap<PC9S_vector2>__FRPC9S_vector2RPC9S_vector2 | T_swap<PC9S_vector2>__FRPC9S_vector2RPC9S_vector2 |
| 0x00436450 | 212 | GLOBAL | I3D_material | {ctor} | __ct__12I3D_materialFv |
| 0x00436530 | 128 | GLOBAL | I3D_material | {dtor} | __dt__12I3D_materialFv |
| 0x004365b0 | 96 | GLOBAL | I3D_material | Release | Release__12I3D_materialFv |
| 0x00436610 | 32 | GLOBAL | I3D_material | SetAmbient | SetAmbient__12I3D_materialFRC8S_vector |
| 0x00436630 | 32 | GLOBAL | I3D_material | SetDiffuse | SetDiffuse__12I3D_materialFRC8S_vector |
| 0x00436650 | 32 | GLOBAL | I3D_material | SetEmissive | SetEmissive__12I3D_materialFRC8S_vector |
| 0x00436670 | 32 | GLOBAL | I3D_material | SetSpecular | SetSpecular__12I3D_materialFRC8S_vector |
| 0x00436690 | 12 | GLOBAL | I3D_material | SetSpecularPower | SetSpecularPower__12I3D_materialFf |
| 0x004366a0 | 12 | GLOBAL | I3D_material | SetAlpha | SetAlpha__12I3D_materialFf |
| 0x004366b0 | 360 | GLOBAL | I3D_material | SetTexture | SetTexture__12I3D_materialFP8ITexture |
| 0x00436820 | 300 | GLOBAL | I3D_material | SetEnvTexture | SetEnvTexture__12I3D_materialFP8ITexture |
| 0x00436950 | 172 | GLOBAL | I3D_material | SetEnvTexture | SetEnvTexture__12I3D_materialFP8ITextureUiUif |
| 0x00436a00 | 272 | GLOBAL | I3D_material | Duplicate | Duplicate__12I3D_materialFP12I3D_material |
| 0x00436b10 | 32 | GLOBAL | I3D_material | SetEnvType | SetEnvType__12I3D_materialFUi |
| 0x00436b30 | 32 | GLOBAL | I3D_material | SetEnvMode | SetEnvMode__12I3D_materialFUi |
| 0x00436b50 | 84 | GLOBAL | I3D_material | SetEnvAlpha | SetEnvAlpha__12I3D_materialFf |
| 0x00436bb0 | 208 | GLOBAL | I3D_anim_material | {ctor} | __ct__17I3D_anim_materialFv |
| 0x00436c80 | 348 | GLOBAL | I3D_anim_material | {dtor} | __dt__17I3D_anim_materialFv |
| 0x00436de0 | 364 | GLOBAL | I3D_anim_material | UpdateTexture | UpdateTexture__17I3D_anim_materialFv |
| 0x00436f50 | 372 | GLOBAL | I3D_anim_material | UpdateEnvTexture | UpdateEnvTexture__17I3D_anim_materialFv |
| 0x004370d0 | 812 | GLOBAL | I3D_anim_material | Duplicate | Duplicate__17I3D_anim_materialFP12I3D_material |
| 0x00437400 | 112 | GLOBAL | I3D_anim_material | SetCurrentFrame | SetCurrentFrame__17I3D_anim_materialFi |
| 0x00437470 | 112 | GLOBAL | I3D_anim_material | SetEnvCurrentFrame | SetEnvCurrentFrame__17I3D_anim_materialFi |
| 0x004374e0 | 48 | GLOBAL | I3D_anim_material | SetTexture | SetTexture__17I3D_anim_materialFP8ITexture |
| 0x00437510 | 48 | GLOBAL | I3D_anim_material | SetEnvTexture | SetEnvTexture__17I3D_anim_materialFP8ITexture |
| 0x00437540 | 740 | GLOBAL | I3D_material | LoadImage | LoadImage__12I3D_materialFv |
| 0x00437830 | 1840 | GLOBAL | I3D_anim_material | LoadImage | LoadImage__17I3D_anim_materialFv |
| 0x00437f60 | 60 | GLOBAL |  | RandFloat | RandFloat |
| 0x00437fa0 | 168 | GLOBAL |  | RotateYMatrix__Ff | RotateYMatrix__Ff |
| 0x00438050 | 168 | GLOBAL |  | RotateZMatrix__Ff | RotateZMatrix__Ff |
| 0x00438100 | 44 | GLOBAL | S_matrix | Identity | Identity__8S_matrixFv |
| 0x00438130 | 632 | GLOBAL | S_matrix | Inverse | Inverse__8S_matrixFRC8S_matrix |
| 0x004383b0 | 68 | GLOBAL | S_matrix | SetDir | SetDir__8S_matrixFRC8S_vector |
| 0x00438400 | 1168 | GLOBAL | S_matrix | SetDir3 | SetDir3__8S_matrixFRC8S_vector |
| 0x00438890 | 68 | GLOBAL | S_matrix | SetDir | SetDir__8S_matrixFRC8S_vectorf |
| 0x004388e0 | 160 | GLOBAL | S_matrix | SetDir3 | SetDir3__8S_matrixFRC8S_vectorf |
| 0x00438980 | 1372 | GLOBAL | S_matrix | SetDir3 | SetDir3__8S_matrixFRC8S_vectorRC8S_vector |
| 0x00438ee0 | 308 | GLOBAL | S_matrix | SetRot3 | SetRot3__8S_matrixFRC6S_quat |
| 0x00439020 | 124 | GLOBAL |  | __amu__8S_matrixFRC8S_matrix | __amu__8S_matrixFRC8S_matrix |
| 0x004390a0 | 172 | GLOBAL | S_matrix | operator* | __ml__8S_matrixCFRC8S_matrix |
| 0x00439150 | 396 | GLOBAL | S_matrix | RotateByMatrix | RotateByMatrix__8S_matrixCFRC8S_matrix |
| 0x004392e0 | 440 | GLOBAL | S_matrix | GetScale | GetScale__8S_matrixCFv |
| 0x004394a0 | 444 | GLOBAL | S_matrix | GetUScale | GetUScale__8S_matrixCFv |
| 0x00439660 | 120 | GLOBAL | S_matrix4 | Make4X4 | Make4X4__9S_matrix4FRC9S_matrix4RC9S_matrix4 |
| 0x004396e0 | 440 | GLOBAL | S_vector | AngleTo | AngleTo__8S_vectorCFRC8S_vector |
| 0x004398a0 | 44 | GLOBAL |  | __amu__8S_vectorFRC8S_matrix | __amu__8S_vectorFRC8S_matrix |
| 0x004398d0 | 64 | GLOBAL | S_vector | operator* | __ml__8S_vectorCFRC8S_matrix |
| 0x00439910 | 128 | GLOBAL | S_vector | RotateByMatrix | RotateByMatrix__8S_vectorCFRC8S_matrix |
| 0x00439990 | 444 | GLOBAL | S_vector | RotateByNormMatrix | RotateByNormMatrix__8S_vectorCFRC8S_matrix |
| 0x00439b50 | 356 | GLOBAL | S_quat | RotationMatrix | RotationMatrix__6S_quatCFv |
| 0x00439cc0 | 336 | GLOBAL | S_quat | Make | Make__6S_quatFRC8S_vectorf |
| 0x00439e10 | 2968 | GLOBAL | S_quat | Make | Make__6S_quatFRC8S_matrix |
| 0x0043a9b0 | 52 | GLOBAL | S_quat | SetDir | SetDir__6S_quatFRC8S_vectorf |
| 0x0043a9f0 | 400 | GLOBAL | S_quat | Inverse | Inverse__6S_quatCFR8S_vectorRf |
| 0x0043ab80 | 396 | GLOBAL | S_quat | Slerp | Slerp__6S_quatCFRC6S_quatfb |
| 0x0043ad10 | 308 | GLOBAL | S_quat | RotateByMatrix | RotateByMatrix__6S_quatCFRC8S_matrix |
| 0x0043ae50 | 332 | GLOBAL | S_quat | Normalize | Normalize__6S_quatFv |
| 0x0043afa0 | 600 | GLOBAL | S_plane | IntersectionLine | IntersectionLine__7S_planeCFRC7S_planeR8S_vectorR8S_vector |
| 0x0043b200 | 340 | GLOBAL |  | LineLineIntersect__FRC8S_vectorRC8S_vectorRC8S_vectorRC8S_vectorRfRf | LineLineIntersect__FRC8S_vectorRC8S_vectorRC8S_vectorRC8S_vectorRfRf |
| 0x0043b360 | 68 | GLOBAL | I3D_face_group | {ctor} | __ct__14I3D_face_groupFv |
| 0x0043b3b0 | 120 | GLOBAL | I3D_face_group | {dtor} | __dt__14I3D_face_groupFv |
| 0x0043b430 | 92 | GLOBAL | I3D_face_group | SetMaterial | SetMaterial__14I3D_face_groupFP12I3D_material |
| 0x0043b490 | 152 | GLOBAL | I3D_face_group | CreateFaces | CreateFaces__14I3D_face_groupFi |
| 0x0043b530 | 92 | GLOBAL | I3D_face_group | GetFaces | GetFaces__14I3D_face_groupCFUi |
| 0x0043b590 | 128 | GLOBAL | I3D_face_group | Duplicate | Duplicate__14I3D_face_groupFP14I3D_face_group |
| 0x0043b610 | 152 | GLOBAL | I3D_mesh_level | {ctor} | __ct__14I3D_mesh_levelFv |
| 0x0043b6b0 | 160 | GLOBAL | I3D_mesh_level | {dtor} | __dt__14I3D_mesh_levelFv |
| 0x0043b750 | 64 | GLOBAL | I3D_mesh_level | Release | Release__14I3D_mesh_levelFv |
| 0x0043b790 | 144 | GLOBAL | I3D_mesh_level | UpdateBBox | UpdateBBox__14I3D_mesh_levelFv |
| 0x0043b820 | 352 | GLOBAL | I3D_mesh_level | SetVertexMesh | SetVertexMesh__14I3D_mesh_levelFP10I3D_vertexi |
| 0x0043b980 | 316 | GLOBAL | I3D_mesh_level | CreateVertexMesh | CreateVertexMesh__14I3D_mesh_levelFi |
| 0x0043bac0 | 92 | GLOBAL | I3D_mesh_level | LockVertices | LockVertices__14I3D_mesh_levelFUi |
| 0x0043bb20 | 16 | GLOBAL | I3D_mesh_level | UnlockVertices | UnlockVertices__14I3D_mesh_levelFv |
| 0x0043bb30 | 544 | GLOBAL | I3D_mesh_level | Duplicate | Duplicate__14I3D_mesh_levelFP14I3D_mesh_level |
| 0x0043bd50 | 72 | GLOBAL | I3D_mesh_level | GetFGroup | GetFGroup__14I3D_mesh_levelFi |
| 0x0043bda0 | 52 | GLOBAL | I3D_mesh_level | NumFaces | NumFaces__14I3D_mesh_levelCFv |
| 0x0043bde0 | 252 | GLOBAL | I3D_mesh_level | AddFGroup | AddFGroup__14I3D_mesh_levelFP14I3D_face_group |
| 0x0043bee0 | 404 | GLOBAL | I3D_mesh_level | CreateVRAMBuffers | CreateVRAMBuffers__14I3D_mesh_levelFUiPC8S_matrixP18COMPLETE_VB_STRUCT |
| 0x0043c080 | 144 | GLOBAL | I3D_mesh_level | DeleteVRAMBuffers | DeleteVRAMBuffers__14I3D_mesh_levelFv |
| 0x0043c110 | 972 | GLOBAL | I3D_mesh_level | UpdateVertices | UpdateVertices__14I3D_mesh_levelFbPC8S_matrixP18COMPLETE_VB_STRUCT |
| 0x0043c4e0 | 80 | GLOBAL | I3D_mesh_level | GetExtraDataStride | GetExtraDataStride__14I3D_mesh_levelFv |
| 0x0043c530 | 152 | GLOBAL | I3D_mesh_level | CreateExtraData | CreateExtraData__14I3D_mesh_levelFUi |
| 0x0043c5d0 | 136 | GLOBAL | I3D_mesh_object | {ctor} | __ct__15I3D_mesh_objectFv |
| 0x0043c660 | 136 | GLOBAL | I3D_mesh_object | {dtor} | __dt__15I3D_mesh_objectFv |
| 0x0043c6f0 | 64 | GLOBAL | I3D_mesh_object | Release | Release__15I3D_mesh_objectFv |
| 0x0043c730 | 20 | GLOBAL | I3D_mesh_object | GetLOD | GetLOD__15I3D_mesh_objectCFi |
| 0x0043c750 | 352 | GLOBAL | I3D_mesh_object | UpdateBoundVolume | UpdateBoundVolume__15I3D_mesh_objectFv |
| 0x0043c8b0 | 228 | GLOBAL | I3D_mesh_object | SetLOD | SetLOD__15I3D_mesh_objectFP14I3D_mesh_leveli |
| 0x0043c9a0 | 404 | GLOBAL | I3D_mesh_object | Duplicate | Duplicate__15I3D_mesh_objectFP15I3D_mesh_object |
| 0x0043cb40 | 88 | GLOBAL | I3D_mirror | BogusConstructor | BogusConstructor__10I3D_mirrorFv |
| 0x0043cba0 | 68 | GLOBAL | I3D_mirror | BogusDestructor | BogusDestructor__10I3D_mirrorFv |
| 0x0043cbf0 | 380 | GLOBAL | I3D_mirror | SetColor | SetColor__10I3D_mirrorFRC8S_vector |
| 0x0043cd70 | 92 | GLOBAL | I3D_mirror | GetFaces | GetFaces__10I3D_mirrorCFUi |
| 0x0043cdd0 | 316 | GLOBAL | I3D_mirror | SetViewBox | SetViewBox__10I3D_mirrorFRC8I3D_bboxRC8S_matrix |
| 0x0043cf10 | 312 | GLOBAL | I3D_mirror | LoadImage | LoadImage__10I3D_mirrorFv |
| 0x0043d050 | 136 | 13 | I3D_mirror | {dtor} | __dt__10I3D_mirrorFv |
| 0x0043d0e0 | 44 | GLOBAL | I3D_model | BogusConstructor | BogusConstructor__9I3D_modelFv |
| 0x0043d110 | 52 | GLOBAL | I3D_model | BogusDestructor | BogusDestructor__9I3D_modelFv |
| 0x0043d150 | 188 | GLOBAL | I3D_model | Close | Close__9I3D_modelFv |
| 0x0043d210 | 440 | GLOBAL | I3D_model | Open | Open__9I3D_modelFPCcUiPF21LS3D_CALLBACK_MESSAGEUiUiPv_vPv |
| 0x0043d3d0 | 216 | GLOBAL | I3D_model | AddFrame | AddFrame__9I3D_modelFP9I3D_frame |
| 0x0043d4b0 | 256 | GLOBAL | I3D_model | DeleteFrame | DeleteFrame__9I3D_modelFP9I3D_frame |
| 0x0043d5b0 | 212 | GLOBAL | I3D_model | SetAnimation | SetAnimation__9I3D_modelFP17I3D_animation_setUif15I3D_ANIMOPTIONS |
| 0x0043d690 | 64 | GLOBAL | I3D_model | GetAnimationSet | GetAnimationSet__9I3D_modelCFUi |
| 0x0043d6d0 | 56 | GLOBAL | I3D_model | StopAnimation | StopAnimation__9I3D_modelFUi |
| 0x0043d710 | 60 | GLOBAL | I3D_model | StartAnimation | StartAnimation__9I3D_modelFUi |
| 0x0043d750 | 144 | GLOBAL | I3D_model | SetAnimBlend | SetAnimBlend__9I3D_modelFUif |
| 0x0043d7e0 | 68 | GLOBAL | I3D_model | GetAnimBlend | GetAnimBlend__9I3D_modelCFUi |
| 0x0043d830 | 96 | GLOBAL | I3D_model | SetAnimTime | SetAnimTime__9I3D_modelFUii |
| 0x0043d890 | 64 | GLOBAL | I3D_model | GetAnimTime | GetAnimTime__9I3D_modelCFUi |
| 0x0043d8d0 | 76 | GLOBAL | I3D_model | SetAnimOptions | SetAnimOptions__9I3D_modelFUi15I3D_ANIMOPTIONS |
| 0x0043d920 | 64 | GLOBAL | I3D_model | GetAnimOptions | GetAnimOptions__9I3D_modelCFUi |
| 0x0043d960 | 64 | GLOBAL | I3D_model | AnimIsOn | AnimIsOn__9I3D_modelCFUi |
| 0x0043d9a0 | 196 | GLOBAL | I3D_model | SetName | SetName__9I3D_modelFv |
| 0x0043da70 | 60 | GLOBAL | I3D_model | SetName | SetName__9I3D_modelFPCc |
| 0x0043dab0 | 840 | GLOBAL |  | duplicateChildren__FP9I3D_frameP9I3D_frame | duplicateChildren__FP9I3D_frameP9I3D_frame |
| 0x0043de00 | 564 | GLOBAL | I3D_model | DuplicateHierarchy | DuplicateHierarchy__9I3D_modelFP9I3D_model |
| 0x0043e040 | 128 | GLOBAL | I3D_model | Duplicate | Duplicate__9I3D_modelFP9I3D_frame |
| 0x0043e0c0 | 80 | GLOBAL | I3D_model | Tick | Tick__9I3D_modelFi |
| 0x0043e110 | 68 | GLOBAL | I3D_model | Tick | Tick__9I3D_modelFPCi |
| 0x0043e160 | 40 | GLOBAL | I3D_model | ResetAllPoses | ResetAllPoses__9I3D_modelFv |
| 0x0043e190 | 32 | GLOBAL | I3D_model | NumInterpolators | NumInterpolators__9I3D_modelCFv |
| 0x0043e1b0 | 32 | GLOBAL | I3D_model | GetInterpolators | GetInterpolators__9I3D_modelCFv |
| 0x0043e1d0 | 140 | GLOBAL | I3D_model | SetPose | SetPose__9I3D_modelFP12I3D_pose_set |
| 0x0043e260 | 156 | GLOBAL | I3D_model | SetPose | SetPose__9I3D_modelFP9I3D_frameP8I3D_pose |
| 0x0043e300 | 156 | GLOBAL | I3D_model | SetPose | SetPose__9I3D_modelFPCcP8I3D_pose |
| 0x0043e3a0 | 52 | GLOBAL | I3D_model | SetPoseBlend | SetPoseBlend__9I3D_modelFf |
| 0x0043e3e0 | 52 | GLOBAL | I3D_model | SetPoseBlend | SetPoseBlend__9I3D_modelFfP12I3D_pose_set |
| 0x0043e420 | 52 | GLOBAL | I3D_model | GetPoseBlend | GetPoseBlend__9I3D_modelCFv |
| 0x0043e460 | 128 | 13 | I3D_model | {dtor} | __dt__9I3D_modelFv |
| 0x0043e4e0 | 92 | 13 | C_list<P10I3D_shadow> | {dtor} | __dt__21C_list<P10I3D_shadow>Fv |
| 0x0043e540 | 92 | 13 | C_list<11TARGET_ITEM> | {dtor} | __dt__21C_list<11TARGET_ITEM>Fv |
| 0x0043e5a0 | 32 | GLOBAL | morphBase | BogusConstructor | BogusConstructor__9morphBaseFv |
| 0x0043e5c0 | 8 | GLOBAL | morphBase | BogusDestructor | BogusDestructor__9morphBaseFv |
| 0x0043e5d0 | 96 | GLOBAL | morphBase | Clear | Clear__9morphBaseFv |
| 0x0043e630 | 52 | GLOBAL | morphBase | SetWeightB | SetWeightB__9morphBaseFiif |
| 0x0043e670 | 32 | GLOBAL | morphBase | GetWeightB | GetWeightB__9morphBaseCFii |
| 0x0043e690 | 276 | GLOBAL | morphBase | NormalizeWeights | NormalizeWeights__9morphBaseFv |
| 0x0043e7b0 | 732 | GLOBAL | morphBase | UpdateMorphLod | UpdateMorphLod__9morphBaseFiP15I3D_mesh_object |
| 0x0043ea90 | 260 | GLOBAL | morphBase | DuplicateMorphDef | DuplicateMorphDef__9morphBaseFP9morphBase |
| 0x0043eba0 | 788 | GLOBAL | morphBase | LoadImageB | LoadImageB__9morphBaseFv |
| 0x0043eec0 | 44 | GLOBAL | I3D_morph | BogusConstructor | BogusConstructor__9I3D_morphFv |
| 0x0043eef0 | 48 | GLOBAL | I3D_morph | BogusDestructor | BogusDestructor__9I3D_morphFv |
| 0x0043ef20 | 8 | GLOBAL | I3D_morph | SetSpace | SetSpace__9I3D_morphFb |
| 0x0043ef30 | 8 | GLOBAL | I3D_morph | NumLevels | NumLevels__9I3D_morphCFv |
| 0x0043ef40 | 8 | GLOBAL | I3D_morph | NumRegions | NumRegions__9I3D_morphCFv |
| 0x0043ef50 | 160 | GLOBAL | I3D_morph | SetWeight | SetWeight__9I3D_morphFiif |
| 0x0043eff0 | 84 | GLOBAL | I3D_morph | GetWeight | GetWeight__9I3D_morphCFii |
| 0x0043f050 | 124 | GLOBAL | I3D_morph | SetMesh | SetMesh__9I3D_morphFP15I3D_mesh_objectb |
| 0x0043f0d0 | 848 | GLOBAL | I3D_morph | Duplicate | Duplicate__9I3D_morphFP9I3D_frame |
| 0x0043f420 | 20 | GLOBAL | I3D_morph | UpdateBoundProc | UpdateBoundProc__9I3D_morphFv |
| 0x0043f440 | 236 | GLOBAL | I3D_morph | m_AddFGroups | m_AddFGroups__9I3D_morphFv |
| 0x0043f530 | 292 | GLOBAL | MorphDef | {ctor} | __ct__8MorphDefFii |
| 0x0043f660 | 92 | GLOBAL | MorphDef | {dtor} | __dt__8MorphDefFv |
| 0x0043f6c0 | 92 | GLOBAL | MorphDef | Release | Release__8MorphDefFv |
| 0x0043f720 | 20 | GLOBAL | morph_region | {ctor} | __ct__12morph_regionFv |
| 0x0043f740 | 92 | GLOBAL | morph_region | {dtor} | __dt__12morph_regionFv |
| 0x0043f7a0 | 152 | GLOBAL | I3D_morph | LoadImage | LoadImage__9I3D_morphFv |
| 0x0043f840 | 176 | 13 | I3D_morph | {dtor} | __dt__9I3D_morphFv |
| 0x0043f8f0 | 8 | 13 | I3D_morph | @288@GetWeight | @288@GetWeight__9I3D_morphCFii |
| 0x0043f900 | 8 | 13 | I3D_morph | @288@SetWeight | @288@SetWeight__9I3D_morphFiif |
| 0x0043f910 | 8 | 13 | I3D_morph | @288@NumRegions | @288@NumRegions__9I3D_morphCFv |
| 0x0043f920 | 8 | 13 | I3D_morph | @288@NumLevels | @288@NumLevels__9I3D_morphCFv |
| 0x0043f930 | 8 | 13 | I3D_morph | @528@BogusDestructor | @528@BogusDestructor__9I3D_morphFv |
| 0x0043f940 | 8 | 13 | I3D_morph | @528@BogusConstructor | @528@BogusConstructor__9I3D_morphFv |
| 0x0043f950 | 100 | GLOBAL | I3D_object | BogusConstructor | BogusConstructor__10I3D_objectFv |
| 0x0043f9c0 | 60 | GLOBAL | I3D_object | BogusDestructor | BogusDestructor__10I3D_objectFv |
| 0x0043fa00 | 316 | GLOBAL | I3D_object | SetMesh | SetMesh__10I3D_objectFP15I3D_mesh_objectb |
| 0x0043fb40 | 212 | GLOBAL | I3D_object | DetachMesh | DetachMesh__10I3D_objectFv |
| 0x0043fc20 | 500 | GLOBAL | I3D_object | Duplicate | Duplicate__10I3D_objectFP9I3D_frame |
| 0x0043fe20 | 1400 | GLOBAL | I3D_object | SetSpace | SetSpace__10I3D_objectFb |
| 0x004403a0 | 8 | GLOBAL | I3D_object | SetLightness | SetLightness__10I3D_objectFf |
| 0x004403b0 | 8 | GLOBAL | I3D_object | SetTransparency | SetTransparency__10I3D_objectFf |
| 0x004403c0 | 108 | GLOBAL | I3D_object | UpdateVertices | UpdateVertices__10I3D_objectFv |
| 0x00440430 | 72 | GLOBAL | I3D_object | UpdateVertices | UpdateVertices__10I3D_objectFi |
| 0x00440480 | 1816 | GLOBAL | I3D_object | m_AddFGroups | m_AddFGroups__10I3D_objectFv |
| 0x00440ba0 | 2756 | GLOBAL | I3D_object | m_RenderFGroup | m_RenderFGroup__10I3D_objectFP21RENDER_FG_HELP_STRUCT |
| 0x00441670 | 96 | GLOBAL | I3D_object | LoadImage | LoadImage__10I3D_objectFv |
| 0x004416d0 | 172 | GLOBAL | I3D_occluder | BogusConstructor | BogusConstructor__12I3D_occluderFv |
| 0x00441780 | 232 | GLOBAL | I3D_occluder | BogusDestructor | BogusDestructor__12I3D_occluderFv |
| 0x00441870 | 260 | GLOBAL | I3D_occluder | AddToRemoveFromSceneRing | AddToRemoveFromSceneRing__12I3D_occluderFP9I3D_scene |
| 0x00441980 | 208 | GLOBAL | I3D_occluder | LinkTo | LinkTo__12I3D_occluderFP9I3D_frameUi |
| 0x00441a50 | 732 | GLOBAL | I3D_occluder | UpdateV | UpdateV__12I3D_occluderFv |
| 0x00441d30 | 140 | GLOBAL | I3D_occluder | Duplicate | Duplicate__12I3D_occluderFP9I3D_frame |
| 0x00441dc0 | 440 | GLOBAL | I3D_occluder | Build | Build__12I3D_occluderFPC8S_vectorUiPCUsUiUi |
| 0x00441f80 | 984 | GLOBAL | I3D_occluder | Build | Build__12I3D_occluderFv |
| 0x00442360 | 1152 | 13 | XSBTree<8TRISIDE2> | Add_r | Add_r__18XSBTree<8TRISIDE2>FPQ218XSBTree<8TRISIDE2>5_NODE |
| 0x004427e0 | 1784 | GLOBAL | I3D_occluder | UpdateShadow | UpdateShadow__12I3D_occluderFv |
| 0x00442ee0 | 224 | GLOBAL | I3D_occluder | TestVertListW | TestVertListW__12I3D_occluderFiP8S_vector |
| 0x00442fc0 | 496 | LOCAL |  | CullOcclsEachOther__FP10I3D_sector | CullOcclsEachOther__FP10I3D_sector |
| 0x004431b0 | 472 | LOCAL |  | POIS_EnumProc__FP10I3D_sector | POIS_EnumProc__FP10I3D_sector |
| 0x00443390 | 104 | GLOBAL |  | ProcessOccludersInSector__FP10I3D_sector | ProcessOccludersInSector__FP10I3D_sector |
| 0x00443400 | 144 | GLOBAL |  | UOL_EnumProc__FP9I3D_frame | UOL_EnumProc__FP9I3D_frame |
| 0x00443490 | 124 | GLOBAL |  | OCCL_UpdateOcclLists__FP9I3D_scene | OCCL_UpdateOcclLists__FP9I3D_scene |
| 0x00443510 | 164 | GLOBAL |  | ProcessOccluders__FP9I3D_scene | ProcessOccluders__FP9I3D_scene |
| 0x004435c0 | 704 | GLOBAL |  | OCCL_PrepareSector2__FP10I3D_sector | OCCL_PrepareSector2__FP10I3D_sector |
| 0x00443880 | 360 | GLOBAL | I3D_occluder | LoadImage | LoadImage__12I3D_occluderFv |
| 0x004439f0 | 96 | 13 | I3D_occluder | {dtor} | __dt__12I3D_occluderFv |
| 0x00443a50 | 532 | GLOBAL |  | InitParticle__Fv | InitParticle__Fv |
| 0x00443c70 | 124 | GLOBAL |  | CloseParticle__Fv | CloseParticle__Fv |
| 0x00443cf0 | 84 | GLOBAL |  | UnlockParticleVB__Fv | UnlockParticleVB__Fv |
| 0x00443d50 | 32 | GLOBAL |  | ResetParticleVBManager__Fv | ResetParticleVBManager__Fv |
| 0x00443d70 | 192 | GLOBAL | I3D_emitor | BogusConstructor | BogusConstructor__10I3D_emitorFv |
| 0x00443e30 | 92 | GLOBAL | I3D_emitor | BogusDestructor | BogusDestructor__10I3D_emitorFv |
| 0x00443e90 | 96 | GLOBAL | I3D_emitor | SetTexture | SetTexture__10I3D_emitorFP8ITexture |
| 0x00443ef0 | 1204 | GLOBAL | I3D_emitor | SetParam | SetParam__10I3D_emitorF14PARTICLE_PARAMUi |
| 0x004443b0 | 492 | GLOBAL | I3D_emitor | GetParam | GetParam__10I3D_emitorF14PARTICLE_PARAM |
| 0x004445a0 | 224 | GLOBAL | I3D_emitor | Reset | Reset__10I3D_emitorFv |
| 0x00444680 | 1864 | GLOBAL | I3D_emitor | AddAnimTexInfo | AddAnimTexInfo__10I3D_emitorFP13ANIM_TEX_INFOi |
| 0x00444dd0 | 348 | 13 | C_list<7UV_RECT> | operator= | __as__16C_list<7UV_RECT>FRC16C_list<7UV_RECT> |
| 0x00444f30 | 28 | 13 | ANIM_TEX_INFO | {ctor} | __ct__13ANIM_TEX_INFOFv |
| 0x00444f50 | 8 | 13 | UV_RECT | {ctor} | __ct__7UV_RECTFv |
| 0x00444f60 | 224 | 13 | C_list<7UV_RECT> | reserve | reserve__16C_list<7UV_RECT>Fi |
| 0x00445040 | 208 | GLOBAL | I3D_emitor | DelAnimTexInfo | DelAnimTexInfo__10I3D_emitorFi |
| 0x00445110 | 40 | GLOBAL | I3D_emitor | SetEmit | SetEmit__10I3D_emitorFb |
| 0x00445140 | 1796 | GLOBAL | ElementStruct | Tick | Tick__13ElementStructFiP10I3D_emitori |
| 0x00445850 | 3600 | GLOBAL | I3D_emitor | Tick | Tick__10I3D_emitorFi |
| 0x00446660 | 256 | 13 | ElementStruct | operator= | __as__13ElementStructFRC13ElementStruct |
| 0x00446760 | 776 | GLOBAL | I3D_emitor | RecomputeBVolume | RecomputeBVolume__10I3D_emitorFv |
| 0x00446a70 | 948 | GLOBAL | I3D_emitor | Duplicate | Duplicate__10I3D_emitorFP9I3D_frame |
| 0x00446e30 | 464 | 13 | C_list<13ANIM_TEX_INFO> | reserve | reserve__23C_list<13ANIM_TEX_INFO>Fi |
| 0x00447000 | 28 | GLOBAL | I3D_emitor | KillElements | KillElements__10I3D_emitorFv |
| 0x00447020 | 540 | GLOBAL | I3D_emitor | LoadImage | LoadImage__10I3D_emitorFv |
| 0x00447240 | 408 | GLOBAL | I3D_emitor | LockVRAMVBuffer | LockVRAMVBuffer__10I3D_emitorFif |
| 0x004473e0 | 2652 | GLOBAL | I3D_emitor | m_AddFGroups | m_AddFGroups__10I3D_emitorFv |
| 0x00447e40 | 1640 | GLOBAL | I3D_emitor | m_RenderFGroup | m_RenderFGroup__10I3D_emitorFP21RENDER_FG_HELP_STRUCT |
| 0x004484b0 | 276 | 13 | I3D_emitor | {dtor} | __dt__10I3D_emitorFv |
| 0x004485d0 | 8 | 13 | I3D_emitor | SetD3DSrcBlend | SetD3DSrcBlend__10I3D_emitorFUi |
| 0x004485e0 | 8 | 13 | I3D_emitor | GetD3DSrcBlend | GetD3DSrcBlend__10I3D_emitorFv |
| 0x004485f0 | 8 | 13 | I3D_emitor | SetD3DDestBlend | SetD3DDestBlend__10I3D_emitorFUi |
| 0x00448600 | 8 | 13 | I3D_emitor | GetD3DDestBlend | GetD3DDestBlend__10I3D_emitorFv |
| 0x00448610 | 92 | 13 | C_sort_list<i> | {dtor} | __dt__14C_sort_list<i>Fv |
| 0x00448670 | 140 | GLOBAL | I3D_portal | {ctor} | __ct__10I3D_portalFv |
| 0x00448700 | 116 | GLOBAL | I3D_portal | {dtor} | __dt__10I3D_portalFv |
| 0x00448780 | 128 | GLOBAL | I3D_portal | Duplicate | Duplicate__10I3D_portalFPC10I3D_portal |
| 0x00448800 | 224 | GLOBAL | I3D_portal | AddToSceneRing | AddToSceneRing__10I3D_portalFP9I3D_scene |
| 0x004488e0 | 812 | GLOBAL | I3D_portal | UpdatePortal | UpdatePortal__10I3D_portalFv |
| 0x00448c10 | 3696 | GLOBAL |  | CTP_EnumProcFirst__FP9I3D_frame | CTP_EnumProcFirst__FP9I3D_frame |
| 0x00449a80 | 2484 | GLOBAL |  | CTP_EnumProc__FP9I3D_frame | CTP_EnumProc__FP9I3D_frame |
| 0x0044a440 | 2072 | GLOBAL |  | CullThruPortal__FP9I3D_frameP10I3D_portalUi | CullThruPortal__FP9I3D_frameP10I3D_portalUi |
| 0x0044ac60 | 8 | GLOBAL |  | CullThruPortal_First__FP9I3D_frameP10I3D_portalUi | CullThruPortal_First__FP9I3D_frameP10I3D_portalUi |
| 0x0044ac70 | 2060 | GLOBAL |  | ParsePortal__FP10I3D_portalP10I3D_portalb | ParsePortal__FP10I3D_portalP10I3D_portalb |
| 0x0044b480 | 1036 | GLOBAL |  | ParsePortalsInSector__FP10I3D_sectorP10I3D_portal | ParsePortalsInSector__FP10I3D_sectorP10I3D_portal |
| 0x0044b890 | 24 | GLOBAL |  | ParsePortalsInSector_Begin__FP10I3D_sector | ParsePortalsInSector_Begin__FP10I3D_sector |
| 0x0044b8b0 | 1812 | GLOBAL |  | UpdatePortalInterconnection__FP9I3D_scene | UpdatePortalInterconnection__FP9I3D_scene |
| 0x0044bfd0 | 156 | GLOBAL |  | ReportBadPortals__FP9I3D_scene | ReportBadPortals__FP9I3D_scene |
| 0x0044c070 | 404 | GLOBAL |  | ProcessPortals__FP9I3D_scene | ProcessPortals__FP9I3D_scene |
| 0x0044c210 | 76 | GLOBAL | I3D_portal | LoadImage | LoadImage__10I3D_portalFv |
| 0x0044c260 | 68 | GLOBAL | I3D_pose | {ctor} | __ct__8I3D_poseFv |
| 0x0044c2b0 | 84 | GLOBAL | I3D_pose | {dtor} | __dt__8I3D_poseFv |
| 0x0044c310 | 92 | GLOBAL | I3D_pose | Release | Release__8I3D_poseFv |
| 0x0044c370 | 52 | GLOBAL | I3D_pose | SetDir | SetDir__8I3D_poseFRC8S_vectorf |
| 0x0044c3b0 | 36 | GLOBAL | I3D_pose_set | {ctor} | __ct__12I3D_pose_setFv |
| 0x0044c3e0 | 204 | GLOBAL | I3D_projector | BogusConstructor | BogusConstructor__13I3D_projectorFv |
| 0x0044c4b0 | 80 | GLOBAL | I3D_projector | BogusDestructor | BogusDestructor__13I3D_projectorFv |
| 0x0044c500 | 508 | GLOBAL | I3D_projector | Duplicate | Duplicate__13I3D_projectorFP9I3D_frame |
| 0x0044c700 | 100 | GLOBAL | I3D_projector | SetTexture | SetTexture__13I3D_projectorFP8ITexture |
| 0x0044c770 | 100 | GLOBAL | I3D_projector | SetMaterial | SetMaterial__13I3D_projectorFP12I3D_material |
| 0x0044c7e0 | 16 | GLOBAL | I3D_projector | SetOrthogonal | SetOrthogonal__13I3D_projectorFb |
| 0x0044c7f0 | 208 | GLOBAL | I3D_projector | SetColor | SetColor__13I3D_projectorFRC8S_vector |
| 0x0044c8c0 | 40 | GLOBAL | I3D_projector | SetAlpha | SetAlpha__13I3D_projectorFf |
| 0x0044c8f0 | 76 | GLOBAL | I3D_projector | SetReverseFactor | SetReverseFactor__13I3D_projectorFf |
| 0x0044c940 | 16 | GLOBAL | I3D_projector | SetMode | SetMode__13I3D_projectorF14PROJECTOR_MODE |
| 0x0044c950 | 16 | GLOBAL | I3D_projector | SetProjectorLock | SetProjectorLock__13I3D_projectorF8I3D_LOCK |
| 0x0044c960 | 164 | GLOBAL | I3D_projector | LoadImage | LoadImage__13I3D_projectorFv |
| 0x0044ca10 | 136 | 13 | I3D_projector | {dtor} | __dt__13I3D_projectorFv |
| 0x0044caa0 | 944 | GLOBAL | I3D_scene | BogusConstructor | BogusConstructor__9I3D_sceneFv |
| 0x0044ce50 | 452 | GLOBAL | I3D_scene | BogusDestructor | BogusDestructor__9I3D_sceneFv |
| 0x0044d020 | 144 | GLOBAL | I3D_scene | m_freeOwnedFrames | m_freeOwnedFrames__9I3D_sceneFv |
| 0x0044d0b0 | 32 | GLOBAL | I3D_scene | SetCallback | SetCallback__9I3D_sceneFPF20SCN_CALLBACK_MESSAGEUiUi_UiUi |
| 0x0044d0d0 | 32 | GLOBAL | I3D_scene | Render | Render__9I3D_sceneFb |
| 0x0044d0f0 | 300 | GLOBAL | I3D_scene | EnumFrames | EnumFrames__9I3D_sceneCFPFP9I3D_frameUi_10I3DENUMRETUiUiPCc |
| 0x0044d220 | 228 | GLOBAL | I3D_scene | FindFrame | FindFrame__9I3D_sceneCFPCcUi |
| 0x0044d310 | 508 | GLOBAL | I3D_scene | AddFrame | AddFrame__9I3D_sceneFP9I3D_frame |
| 0x0044d510 | 432 | GLOBAL | I3D_scene | DeleteFrame | DeleteFrame__9I3D_sceneFP9I3D_frame |
| 0x0044d6c0 | 104 | GLOBAL | I3D_scene | SetGlobalColor | SetGlobalColor__9I3D_sceneFRC8S_vector |
| 0x0044d730 | 1640 | GLOBAL | I3D_scene | GetLightness | GetLightness__9I3D_sceneFRC8S_vectorP10I3D_sectorUiP9I3D_frame |
| 0x0044dda0 | 208 | GLOBAL | I3D_scene | SetActiveCamera | SetActiveCamera__9I3D_sceneFP10I3D_camera |
| 0x0044de70 | 836 | GLOBAL | I3D_scene | UnmapScreenPoint | UnmapScreenPoint__9I3D_sceneCFiiR8S_vectorR8S_vector |
| 0x0044e1c0 | 424 | GLOBAL | I3D_scene | TransformPoints | TransformPoints__9I3D_sceneCFPC8S_vectorP9S_vector4Ui |
| 0x0044e370 | 224 | GLOBAL | I3D_scene | AddTickFrame | AddTickFrame__9I3D_sceneFP9I3D_frame |
| 0x0044e450 | 228 | GLOBAL | I3D_scene | DeleteTickFrame | DeleteTickFrame__9I3D_sceneFP9I3D_frame |
| 0x0044e540 | 256 | GLOBAL | I3D_scene | Tick | Tick__9I3D_sceneFi |
| 0x0044e640 | 192 | GLOBAL | I3D_scene | CreateSector | CreateSector__9I3D_sceneFv |
| 0x0044e700 | 136 | GLOBAL | I3D_scene | GetSector | GetSector__9I3D_sceneCFRC8S_vector |
| 0x0044e790 | 208 | GLOBAL | I3D_scene | GetSector | GetSector__9I3D_sceneCFRC8S_vectorP10I3D_sector |
| 0x0044e860 | 268 | GLOBAL | I3D_scene | GetSector | GetSector__9I3D_sceneCFRC8S_vectorP10I3D_sectorRf |
| 0x0044e970 | 144 | GLOBAL | I3D_scene | GetSector | GetSector__9I3D_sceneCFRC8S_vectorPP10I3D_sectori |
| 0x0044ea00 | 220 | GLOBAL | I3D_scene | SetFrameSectorPos | SetFrameSectorPos__9I3D_sceneCFP9I3D_frameRC8S_vector |
| 0x0044eae0 | 48 | GLOBAL | I3D_scene | SetFrameSectorPos | SetFrameSectorPos__9I3D_sceneCFP9I3D_frameRC8S_vectorRC8S_vector |
| 0x0044eb10 | 172 | GLOBAL | I3D_scene | Open | Open__9I3D_sceneFPCcUiPF21LS3D_CALLBACK_MESSAGEUiUiPv_vPv |
| 0x0044ebc0 | 444 | GLOBAL | I3D_scene | Close | Close__9I3D_sceneFv |
| 0x0044ed80 | 152 | GLOBAL | I3D_scene | SetBackdropRange | SetBackdropRange__9I3D_sceneFff |
| 0x0044ee20 | 120 | GLOBAL | I3D_scene | MakeReadySectors | MakeReadySectors__9I3D_sceneFv |
| 0x0044eea0 | 32 | 13 | I3D_scene::EnumHelp$1596I3d_scene_cpp | I3D_EnumProc | I3D_EnumProc__Q29I3D_scene26EnumHelp$1596I3d_scene_cppFP9I3D_frameUi |
| 0x0044eec0 | 216 | LOCAL |  | sceneSS__FP9I3D_frame | sceneSS__FP9I3D_frame |
| 0x0044efa0 | 12 | GLOBAL | I3D_scene | SetSpace | SetSpace__9I3D_sceneFb |
| 0x0044efb0 | 168 | GLOBAL | I3D_scene | SetViewport | SetViewport__9I3D_sceneFUiUiUiUi |
| 0x0044f060 | 120 | GLOBAL | I3D_scene | SetViewProjMatrix | SetViewProjMatrix__9I3D_sceneFRC9S_matrix4 |
| 0x0044f0e0 | 388 | GLOBAL | I3D_scene | m_UpdateSoundProperties | m_UpdateSoundProperties__9I3D_sceneFP10I3D_sectorP10I3D_sectorf |
| 0x0044f270 | 212 | GLOBAL | I3D_scene | SetAnimation | SetAnimation__9I3D_sceneFP17I3D_animation_setUif15I3D_ANIMOPTIONS |
| 0x0044f350 | 64 | GLOBAL | I3D_scene | GetAnimationSet | GetAnimationSet__9I3D_sceneCFUi |
| 0x0044f390 | 56 | GLOBAL | I3D_scene | StopAnimation | StopAnimation__9I3D_sceneFUi |
| 0x0044f3d0 | 144 | GLOBAL | I3D_scene | SetAnimBlend | SetAnimBlend__9I3D_sceneFUif |
| 0x0044f460 | 68 | GLOBAL | I3D_scene | GetAnimBlend | GetAnimBlend__9I3D_sceneCFUi |
| 0x0044f4b0 | 96 | GLOBAL | I3D_scene | SetAnimTime | SetAnimTime__9I3D_sceneFUii |
| 0x0044f510 | 64 | GLOBAL | I3D_scene | GetAnimTime | GetAnimTime__9I3D_sceneCFUi |
| 0x0044f550 | 64 | GLOBAL | I3D_scene | AnimIsOn | AnimIsOn__9I3D_sceneCFUi |
| 0x0044f590 | 32 | GLOBAL | I3D_scene | NumInterpolators | NumInterpolators__9I3D_sceneCFv |
| 0x0044f5b0 | 32 | GLOBAL | I3D_scene | GetInterpolators | GetInterpolators__9I3D_sceneCFv |
| 0x0044f5d0 | 288 | GLOBAL | I3D_scene | WeatherSystemReset | WeatherSystemReset__9I3D_sceneFv |
| 0x0044f6f0 | 1272 | GLOBAL |  | m_PrepareDummyStruct__FP12DUMMY_STRUCTP9I3D_dummy | m_PrepareDummyStruct__FP12DUMMY_STRUCTP9I3D_dummy |
| 0x0044fbf0 | 744 | GLOBAL | I3D_scene | WeatherSystemAddDummy | WeatherSystemAddDummy__9I3D_sceneFP9I3D_dummy |
| 0x0044fee0 | 48 | 13 | DUMMY_STRUCT | {ctor} | __ct__12DUMMY_STRUCTFv |
| 0x0044ff10 | 288 | GLOBAL | I3D_scene | WeatherSystemDelDummy | WeatherSystemDelDummy__9I3D_sceneFP9I3D_dummy |
| 0x00450030 | 8 | GLOBAL | I3D_scene | WeatherSystemGetDummyCnt | WeatherSystemGetDummyCnt__9I3D_sceneFv |
| 0x00450040 | 8 | GLOBAL | I3D_scene | WeatherSystemGetDumies | WeatherSystemGetDumies__9I3D_sceneFv |
| 0x00450050 | 1340 | GLOBAL | I3D_scene | SetWeatherSystemParam | SetWeatherSystemParam__9I3D_sceneF8WS_PARAMUi |
| 0x00450590 | 332 | GLOBAL | I3D_scene | GetWeatherSystemParam | GetWeatherSystemParam__9I3D_sceneF8WS_PARAM |
| 0x004506e0 | 6652 | GLOBAL | I3D_scene | m_UpdateWeatherSystem | m_UpdateWeatherSystem__9I3D_sceneFv |
| 0x004520e0 | 48 | 13 | WE_STRUCT | {ctor} | __ct__9WE_STRUCTFv |
| 0x00452110 | 3096 | GLOBAL | I3D_scene | m_RenderWeatherSystem | m_RenderWeatherSystem__9I3D_sceneFv |
| 0x00452d30 | 520 | GLOBAL | I3D_scene | TestVisibility | TestVisibility__9I3D_sceneFP9I3D_frameP10I3D_camera |
| 0x00452f40 | 80 | GLOBAL | I3D_scene | UpdatePortals | UpdatePortals__9I3D_sceneFv |
| 0x00452f90 | 720 | GLOBAL | I3D_scene | AddWatchedFrame | AddWatchedFrame__9I3D_sceneFP9I3D_frameP9I3D_frameP9I3D_frame |
| 0x00453260 | 224 | GLOBAL | I3D_scene | DeleteAllWatchedFrames | DeleteAllWatchedFrames__9I3D_sceneFv |
| 0x00453340 | 380 | GLOBAL | I3D_scene | DeleteWatchedFrame | DeleteWatchedFrame__9I3D_sceneFP9I3D_frame |
| 0x004534c0 | 24 | GLOBAL | I3D_scene | MarkWatchedFrame | MarkWatchedFrame__9I3D_sceneFP9I3D_frame |
| 0x004534e0 | 536 | GLOBAL | I3D_scene | ProcessWatchedFrames | ProcessWatchedFrames__9I3D_sceneFP10I3D_sector |
| 0x00453700 | 312 | 13 | I3D_scene | {dtor} | __dt__9I3D_sceneFv |
| 0x00453840 | 8 | 13 | I3D_scene | SetCallback | SetCallback__9I3D_sceneFPFP9I3D_frame21LS3D_CALLBACK_MESSAGEUiUi_UiUi |
| 0x00453850 | 8 | 13 | I3D_scene | SetDir | SetDir__9I3D_sceneFRC8S_vectorf |
| 0x00453860 | 8 | 13 | I3D_scene | Update | Update__9I3D_sceneFv |
| 0x00453870 | 8 | 13 | I3D_scene | SetOn | SetOn__9I3D_sceneFb |
| 0x00453880 | 8 | 13 | I3D_scene | SetName | SetName__9I3D_sceneFPCc |
| 0x00453890 | 8 | 13 | I3D_scene | Duplicate | Duplicate__9I3D_sceneFP9I3D_frame |
| 0x004538a0 | 12 | 13 | I3D_scene | LinkTo | LinkTo__9I3D_sceneFP9I3D_frameUi |
| 0x004538b0 | 92 | 13 | C_list<P12DUMMY_STRUCT> | {dtor} | __dt__23C_list<P12DUMMY_STRUCT>Fv |
| 0x00453910 | 76 | GLOBAL | S_sector_sound_properties | SetupDefaults | SetupDefaults__25S_sector_sound_propertiesFv |
| 0x00453960 | 852 | GLOBAL | S_sector_sound_properties | MakeBlend | MakeBlend__25S_sector_sound_propertiesFRC25S_sector_sound_propertiesRC25S_sector_sound_propertiesfb |
| 0x00453cc0 | 184 | GLOBAL | I3D_sector | BogusConstructor | BogusConstructor__10I3D_sectorFv |
| 0x00453d80 | 376 | GLOBAL | I3D_sector | BogusDestructor | BogusDestructor__10I3D_sectorFv |
| 0x00453f00 | 36 | GLOBAL | I3D_sector | SetBBox | SetBBox__10I3D_sectorFRC8I3D_bbox |
| 0x00453f30 | 376 | GLOBAL | I3D_sector | Clear | Clear__10I3D_sectorFv |
| 0x004540b0 | 84 | GLOBAL | I3D_sector | SetName | SetName__10I3D_sectorFPCc |
| 0x00454110 | 368 | GLOBAL | I3D_sector | AddToRemoveFromSceneRing | AddToRemoveFromSceneRing__10I3D_sectorFP9I3D_scene |
| 0x00454280 | 572 | GLOBAL | I3D_sector | AddLight | AddLight__10I3D_sectorFP9I3D_light |
| 0x004544c0 | 600 | GLOBAL | I3D_sector | DeleteLight | DeleteLight__10I3D_sectorFP9I3D_light |
| 0x00454720 | 216 | GLOBAL | I3D_sector | UpdateV | UpdateV__10I3D_sectorFv |
| 0x00454800 | 604 | GLOBAL | I3D_sector | UpdateCHPlanesW | UpdateCHPlanesW__10I3D_sectorFv |
| 0x00454a60 | 472 | GLOBAL | I3D_sector | MakeReady | MakeReady__10I3D_sectorFv |
| 0x00454c40 | 340 | GLOBAL | I3D_sector | CheckPoint | CheckPoint__10I3D_sectorFRC8S_vector |
| 0x00454da0 | 372 | GLOBAL | I3D_sector | CheckPointDepth | CheckPointDepth__10I3D_sectorFRC8S_vector |
| 0x00454f20 | 60 | GLOBAL | I3D_sector | SetSoundData | SetSoundData__10I3D_sectorF27I3D_SECTOR_SOUND_DATA_INDEXUi |
| 0x00454f60 | 72 | GLOBAL | I3D_sector | GetSoundData | GetSoundData__10I3D_sectorCF27I3D_SECTOR_SOUND_DATA_INDEX |
| 0x00454fb0 | 496 | GLOBAL | I3D_sector | AddSound | AddSound__10I3D_sectorFP9I3D_sound |
| 0x004551a0 | 528 | GLOBAL | I3D_sector | DeleteSound | DeleteSound__10I3D_sectorFP9I3D_sound |
| 0x004553b0 | 864 | GLOBAL | I3D_sector | Duplicate | Duplicate__10I3D_sectorFP9I3D_frame |
| 0x00455710 | 760 | GLOBAL | I3D_sector | LoadImage | LoadImage__10I3D_sectorFv |
| 0x00455a10 | 20 | GLOBAL | I3D_sector | SetGroupBits | SetGroupBits__10I3D_sectorFUi |
| 0x00455a30 | 224 | 13 | I3D_sector | {dtor} | __dt__10I3D_sectorFv |
| 0x00455b10 | 392 | GLOBAL | I3D_shadow | BogusConstructor | BogusConstructor__10I3D_shadowFv |
| 0x00455ca0 | 224 | GLOBAL | I3D_shadow | BogusDestructor | BogusDestructor__10I3D_shadowFv |
| 0x00455d80 | 240 | GLOBAL | I3D_shadow | Duplicate | Duplicate__10I3D_shadowFP9I3D_frame |
| 0x00455e70 | 8 | GLOBAL | I3D_shadow | SetTo | SetTo__10I3D_shadowFP9I3D_frame |
| 0x00455e80 | 76 | GLOBAL | I3D_shadow | LinkTo | LinkTo__10I3D_shadowFP9I3D_frameUi |
| 0x00455ed0 | 20 | GLOBAL | I3D_shadow | SetShadowLock | SetShadowLock__10I3D_shadowF8I3D_LOCK |
| 0x00455ef0 | 8 | GLOBAL | I3D_shadow | UpdateV | UpdateV__10I3D_shadowFv |
| 0x00455f00 | 120 | GLOBAL | I3D_shadow | DeleteFromAnyShadow | DeleteFromAnyShadow__10I3D_shadowFP9I3D_frame |
| 0x00455f80 | 8 | GLOBAL | I3D_shadow | Close | Close__10I3D_shadowFv |
| 0x00455f90 | 8 | GLOBAL | I3D_shadow | Open | Open__10I3D_shadowFPCcUi |
| 0x00455fa0 | 48 | GLOBAL | I3D_shadow | SetRange | SetRange__10I3D_shadowFf |
| 0x00455fd0 | 104 | GLOBAL | I3D_shadow | SetMaxTextureDistance | SetMaxTextureDistance__10I3D_shadowFf |
| 0x00456040 | 212 | GLOBAL | I3D_shadow | SetColor | SetColor__10I3D_shadowFRC8S_vector |
| 0x00456120 | 72 | GLOBAL | I3D_shadow | SetReverseFactor | SetReverseFactor__10I3D_shadowFf |
| 0x00456170 | 32 | GLOBAL | I3D_shadow | SetOrthogonal | SetOrthogonal__10I3D_shadowFb |
| 0x00456190 | 144 | GLOBAL | I3D_shadow | SetTextureSize | SetTextureSize__10I3D_shadowFii |
| 0x00456220 | 96 | 13 | I3D_shadow | {dtor} | __dt__10I3D_shadowFv |
| 0x00456280 | 76 | GLOBAL | I3D_singlemesh | BogusConstructor | BogusConstructor__14I3D_singlemeshFv |
| 0x004562d0 | 176 | GLOBAL | I3D_singlemesh | BogusDestructor | BogusDestructor__14I3D_singlemeshFv |
| 0x00456380 | 148 | 13 | SMeshDef | {dtor} | __dt__8SMeshDefFv |
| 0x00456420 | 32 | GLOBAL | I3D_singlemesh | UpdateVertices | UpdateVertices__14I3D_singlemeshFv |
| 0x00456440 | 32 | GLOBAL | I3D_singlemesh | UpdateVertices | UpdateVertices__14I3D_singlemeshFi |
| 0x00456460 | 124 | GLOBAL | I3D_singlemesh | GetJointBBox | GetJointBBox__14I3D_singlemeshCFi |
| 0x004564e0 | 100 | GLOBAL | I3D_singlemesh | DeleteVRAMBuffers | DeleteVRAMBuffers__14I3D_singlemeshFi |
| 0x00456550 | 120 | GLOBAL | I3D_singlemesh | DeleteVRAMBuffers | DeleteVRAMBuffers__14I3D_singlemeshFv |
| 0x004565d0 | 8 | GLOBAL | I3D_singlemesh | SetSpace | SetSpace__14I3D_singlemeshFb |
| 0x004565e0 | 44 | GLOBAL | I3D_singlemesh | DuplicateSinglemeshDef | DuplicateSinglemeshDef__14I3D_singlemeshFP14I3D_singlemesh |
| 0x00456610 | 720 | GLOBAL | I3D_singlemesh | Duplicate | Duplicate__14I3D_singlemeshFP9I3D_frame |
| 0x004568e0 | 68 | GLOBAL | I3D_singlemesh | UpdateBoundProc | UpdateBoundProc__14I3D_singlemeshFv |
| 0x00456930 | 912 | GLOBAL | I3D_singlemesh | UpdateBVolumeProc | UpdateBVolumeProc__14I3D_singlemeshFv |
| 0x00456cc0 | 356 | 13 | I3D_joint | GetJointMatrix | GetJointMatrix__9I3D_jointFv |
| 0x00456e30 | 200 | LOCAL |  | findJoint__FP9I3D_frame | findJoint__FP9I3D_frame |
| 0x00456f00 | 380 | GLOBAL | I3D_singlemesh | DBuild | DBuild__14I3D_singlemeshFv |
| 0x00457080 | 132 | GLOBAL | I3D_singlemesh | GetVertexJoint | GetVertexJoint__14I3D_singlemeshFii |
| 0x00457110 | 232 | GLOBAL | I3D_singlemesh | UpdateMList | UpdateMList__14I3D_singlemeshFv |
| 0x00457200 | 1064 | GLOBAL | I3D_singlemesh | GetVertexPos | GetVertexPos__14I3D_singlemeshFi |
| 0x00457630 | 124 | GLOBAL | I3D_singlemesh | m_AddFGroups | m_AddFGroups__14I3D_singlemeshFv |
| 0x004576b0 | 776 | GLOBAL | I3D_singlemesh | LoadImage | LoadImage__14I3D_singlemeshFv |
| 0x004579c0 | 8 | 13 | SM_INFO | {ctor} | __ct__7SM_INFOFv |
| 0x004579d0 | 48 | GLOBAL |  | AtExitSinglemesh__Fv | AtExitSinglemesh__Fv |
| 0x00457a00 | 44 | GLOBAL | I3D_singlemorph | BogusConstructor | BogusConstructor__15I3D_singlemorphFv |
| 0x00457a30 | 48 | GLOBAL | I3D_singlemorph | BogusDestructor | BogusDestructor__15I3D_singlemorphFv |
| 0x00457a60 | 8 | GLOBAL | I3D_singlemorph | NumLevels | NumLevels__15I3D_singlemorphCFv |
| 0x00457a70 | 8 | GLOBAL | I3D_singlemorph | NumRegions | NumRegions__15I3D_singlemorphCFv |
| 0x00457a80 | 148 | GLOBAL | I3D_singlemorph | SetWeight | SetWeight__15I3D_singlemorphFiif |
| 0x00457b20 | 84 | GLOBAL | I3D_singlemorph | GetWeight | GetWeight__15I3D_singlemorphCFii |
| 0x00457b80 | 124 | GLOBAL | I3D_singlemorph | SetMesh | SetMesh__15I3D_singlemorphFP15I3D_mesh_objectb |
| 0x00457c00 | 1344 | GLOBAL | I3D_singlemorph | Duplicate | Duplicate__15I3D_singlemorphFP9I3D_frame |
| 0x00458140 | 8 | GLOBAL | I3D_singlemorph | UpdateBoundProc | UpdateBoundProc__15I3D_singlemorphFv |
| 0x00458150 | 68 | GLOBAL | I3D_singlemorph | GetVertexPos | GetVertexPos__15I3D_singlemorphFi |
| 0x004581a0 | 276 | GLOBAL | I3D_singlemorph | m_AddFGroups | m_AddFGroups__15I3D_singlemorphFv |
| 0x004582c0 | 164 | GLOBAL | I3D_singlemorph | LoadImage | LoadImage__15I3D_singlemorphFv |
| 0x00458370 | 176 | 13 | I3D_singlemorph | {dtor} | __dt__15I3D_singlemorphFv |
| 0x00458420 | 8 | 13 | I3D_singlemorph | @288@GetWeight | @288@GetWeight__15I3D_singlemorphCFii |
| 0x00458430 | 8 | 13 | I3D_singlemorph | @288@SetWeight | @288@SetWeight__15I3D_singlemorphFiif |
| 0x00458440 | 8 | 13 | I3D_singlemorph | @288@NumRegions | @288@NumRegions__15I3D_singlemorphCFv |
| 0x00458450 | 8 | 13 | I3D_singlemorph | @288@NumLevels | @288@NumLevels__15I3D_singlemorphCFv |
| 0x00458460 | 8 | 13 | I3D_singlemorph | @560@BogusDestructor | @560@BogusDestructor__15I3D_singlemorphFv |
| 0x00458470 | 8 | 13 | I3D_singlemorph | @560@BogusConstructor | @560@BogusConstructor__15I3D_singlemorphFv |
| 0x00458480 | 432 | GLOBAL | I3D_sound | BogusConstructor | BogusConstructor__9I3D_soundFv |
| 0x00458630 | 392 | GLOBAL | I3D_sound | BogusDestructor | BogusDestructor__9I3D_soundFv |
| 0x004587c0 | 1120 | GLOBAL | I3D_sound | Open | Open__9I3D_soundFPCcUi |
| 0x00458c20 | 384 | GLOBAL | I3D_sound | IsPlaying | IsPlaying__9I3D_soundCFb |
| 0x00458da0 | 292 | GLOBAL | I3D_sound | SetSoundType | SetSoundType__9I3D_soundF13I3D_SOUNDTYPE |
| 0x00458ed0 | 260 | GLOBAL | I3D_sound | SetRange | SetRange__9I3D_soundFffff |
| 0x00458fe0 | 260 | GLOBAL | I3D_sound | SetCone | SetCone__9I3D_soundFff |
| 0x004590f0 | 80 | GLOBAL | I3D_sound | SetOutVol | SetOutVol__9I3D_soundFf |
| 0x00459140 | 80 | GLOBAL | I3D_sound | SetVolume | SetVolume__9I3D_soundFf |
| 0x00459190 | 100 | GLOBAL | I3D_sound | SetLoop | SetLoop__9I3D_soundFb |
| 0x00459200 | 144 | GLOBAL | I3D_sound | SetCurrTime | SetCurrTime__9I3D_soundFUi |
| 0x00459290 | 136 | GLOBAL | I3D_sound | SetFrequency | SetFrequency__9I3D_soundFf |
| 0x00459320 | 204 | GLOBAL | I3D_sound | SetVelocity | SetVelocity__9I3D_soundFRC8S_vector |
| 0x004593f0 | 36 | GLOBAL | I3D_sound | GetRange | GetRange__9I3D_soundCFRfRfRfRf |
| 0x00459420 | 20 | GLOBAL | I3D_sound | GetCone | GetCone__9I3D_soundCFRfRf |
| 0x00459440 | 56 | GLOBAL | I3D_sound | GetCurrTime | GetCurrTime__9I3D_soundCFv |
| 0x00459480 | 48 | GLOBAL | I3D_sound | GetPlayTime | GetPlayTime__9I3D_soundCFv |
| 0x004594b0 | 832 | GLOBAL | I3D_sound | SetOn | SetOn__9I3D_soundFbb |
| 0x004597f0 | 124 | 13 | I3D_sound | m_Pause | m_Pause__9I3D_soundFv |
| 0x00459870 | 8 | GLOBAL | I3D_sound | ForceUpdate | ForceUpdate__9I3D_soundFv |
| 0x00459880 | 5256 | GLOBAL | I3D_sound | m_UpdateSound | m_UpdateSound__9I3D_soundFv |
| 0x0045ad10 | 312 | GLOBAL | I3D_sound | Duplicate | Duplicate__9I3D_soundFP9I3D_frame |
| 0x0045ae50 | 8 | GLOBAL | I3D_sound | m_SetState | m_SetState__9I3D_soundFUc |
| 0x0045ae60 | 8 | GLOBAL | I3D_sound | LoadImage | LoadImage__9I3D_soundFv |
| 0x0045ae70 | 8 | GLOBAL | I3D_sound | LSI_Lock | LSI_Lock__9I3D_soundFi |
| 0x0045ae80 | 160 | 13 | I3D_sound | {dtor} | __dt__9I3D_soundFv |
| 0x0045af20 | 236 | GLOBAL | I3D_target | BogusConstructor | BogusConstructor__10I3D_targetFv |
| 0x0045b010 | 228 | GLOBAL | I3D_target | BogusDestructor | BogusDestructor__10I3D_targetFv |
| 0x0045b100 | 112 | GLOBAL | I3D_target | Duplicate | Duplicate__10I3D_targetFP9I3D_frame |
| 0x0045b170 | 8 | GLOBAL | I3D_target | SetTargetFlags | SetTargetFlags__10I3D_targetFUi |
| 0x0045b180 | 232 | GLOBAL | I3D_target | AddToTarget | AddToTarget__10I3D_targetFP9I3D_frame |
| 0x0045b270 | 236 | GLOBAL | I3D_target | DeleteFromTarget | DeleteFromTarget__10I3D_targetFP9I3D_frame |
| 0x0045b360 | 100 | GLOBAL | I3D_target | GetTarget | GetTarget__10I3D_targetFP9I3D_frame |
| 0x0045b3d0 | 124 | GLOBAL | I3D_target | DeleteFromAnyTarget | DeleteFromAnyTarget__10I3D_targetFP9I3D_frame |
| 0x0045b450 | 252 | GLOBAL | I3D_target | UpdateV | UpdateV__10I3D_targetFv |
| 0x0045b550 | 1340 | GLOBAL | I3D_target | UpdateFrames | UpdateFrames__10I3D_targetFv |
| 0x0045ba90 | 592 | GLOBAL | I3D_target | LoadImage | LoadImage__10I3D_targetFv |
| 0x0045bce0 | 128 | 13 | I3D_target | {dtor} | __dt__10I3D_targetFv |
| 0x0045bd60 | 40 | GLOBAL | I3D_user | BogusConstructor | BogusConstructor__8I3D_userFv |
| 0x0045bd90 | 68 | GLOBAL | I3D_user | Duplicate | Duplicate__8I3D_userFP9I3D_frame |
| 0x0045bde0 | 96 | 13 | I3D_user | {dtor} | __dt__8I3D_userFv |
| 0x0045be40 | 68 | GLOBAL |  | VisualType2Int__F15I3D_VISUAL_TYPE | VisualType2Int__F15I3D_VISUAL_TYPE |
| 0x0045be90 | 192 | GLOBAL | I3D_visual | BogusConstructor | BogusConstructor__10I3D_visualFv |
| 0x0045bf50 | 28 | GLOBAL | I3D_visual | GetVisualType | GetVisualType__10I3D_visualCFv |
| 0x0045bf70 | 60 | GLOBAL | I3D_visual | UpdateV | UpdateV__10I3D_visualFv |
| 0x0045bfb0 | 8 | GLOBAL | I3D_visual | SetSpace | SetSpace__10I3D_visualFb |
| 0x0045bfc0 | 104 | GLOBAL | I3D_visual | SetShadowMode | SetShadowMode__10I3D_visualF11SHADOW_MODE |
| 0x0045c030 | 36 | GLOBAL | I3D_visual | GetShadowMode | GetShadowMode__10I3D_visualCFv |
| 0x0045c060 | 104 | GLOBAL | I3D_visual | SetProjectorMode | SetProjectorMode__10I3D_visualF11SHADOW_MODE |
| 0x0045c0d0 | 36 | GLOBAL | I3D_visual | GetProjectorMode | GetProjectorMode__10I3D_visualCFv |
| 0x0045c100 | 20 | GLOBAL | I3D_visual | UpdateBoundProc | UpdateBoundProc__10I3D_visualFv |
| 0x0045c120 | 672 | GLOBAL | I3D_visual | UpdateWBoundProc | UpdateWBoundProc__10I3D_visualFv |
| 0x0045c3c0 | 96 | GLOBAL | I3D_visual | SetBoundVolume | SetBoundVolume__10I3D_visualFRC11I3D_bvolume |
| 0x0045c420 | 344 | GLOBAL | I3D_visual | SetBoundBox | SetBoundBox__10I3D_visualFRC8I3D_bbox |
| 0x0045c580 | 8 | GLOBAL | I3D_void_interface | NumLevels | NumLevels__18I3D_void_interfaceCFv |
| 0x0045c590 | 8 | GLOBAL | I3D_void_interface | NumRegions | NumRegions__18I3D_void_interfaceCFv |
| 0x0045c5a0 | 12 | GLOBAL | I3D_void_interface | SetWeight | SetWeight__18I3D_void_interfaceFiif |
| 0x0045c5b0 | 12 | GLOBAL | I3D_void_interface | GetWeight | GetWeight__18I3D_void_interfaceCFii |
| 0x0045c5c0 | 88 | GLOBAL | I3D_volume | BogusConstructor | BogusConstructor__10I3D_volumeFv |
| 0x0045c620 | 8 | GLOBAL | I3D_volume | GetVolumeType | GetVolumeType__10I3D_volumeCFv |
| 0x0045c630 | 492 | GLOBAL | I3D_volume | UpdateV | UpdateV__10I3D_volumeFv |
| 0x0045c820 | 96 | 13 | I3D_volume | {dtor} | __dt__10I3D_volumeFv |
| 0x0045c880 | 272 | GLOBAL |  | FreeTexBlList__FPi | FreeTexBlList__FPi |
| 0x0045c990 | 108 | 13 | TEXBL_STRUCT | {dtor} | __dt__12TEXBL_STRUCTFv |
| 0x0045ca00 | 1416 | GLOBAL | LightMap | m_ConvertRGB | m_ConvertRGB__8LightMapFUsUsPvPPv |
| 0x0045cf90 | 56 | GLOBAL | LightMap | {ctor} | __ct__8LightMapFv |
| 0x0045cfd0 | 2072 | GLOBAL | LightMap | {dtor} | __dt__8LightMapFv |
| 0x0045d7f0 | 28 | 13 | TEXBL_STRUCT | {ctor} | __ct__12TEXBL_STRUCTFv |
| 0x0045d810 | 452 | 13 | C_list<12TEXBL_STRUCT> | push_back | push_back__22C_list<12TEXBL_STRUCT>Fv |
| 0x0045d9e0 | 184 | 13 | C_list<13OFFSET_STRUCT> | reserve | reserve__23C_list<13OFFSET_STRUCT>Fi |
| 0x0045daa0 | 628 | 13 | C_list<12TEXBL_STRUCT> | push_back | push_back__22C_list<12TEXBL_STRUCT>FRC12TEXBL_STRUCT |
| 0x0045dd20 | 304 | 13 | C_list<13OFFSET_STRUCT> | operator= | __as__23C_list<13OFFSET_STRUCT>FRC23C_list<13OFFSET_STRUCT> |
| 0x0045de50 | 380 | GLOBAL | LightMap | m_AddRectToRectList | m_AddRectToRectList__8LightMapFP12TEXBL_STRUCTP5_RECT |
| 0x0045dfd0 | 324 | GLOBAL | LightMap | m_ReplaceRectFromRectList | m_ReplaceRectFromRectList__8LightMapFP12TEXBL_STRUCTP11RECT_STRUCT |
| 0x0045e120 | 88 | GLOBAL | LightMap | m_RemoveRectFromRectList | m_RemoveRectFromRectList__8LightMapFP12TEXBL_STRUCTP11RECT_STRUCT |
| 0x0045e180 | 104 | GLOBAL | LightMap | m_EraseRectList | m_EraseRectList__8LightMapFP12TEXBL_STRUCT |
| 0x0045e1f0 | 228 | GLOBAL | LightMap | m_DuplicateRectList | m_DuplicateRectList__8LightMapFP12TEXBL_STRUCTP12TEXBL_STRUCT |
| 0x0045e2e0 | 2220 | GLOBAL | LightMap | m_AddTexture | m_AddTexture__8LightMapFP12TEXBL_STRUCTP14RECTANGLE_INFO |
| 0x0045eb90 | 76 | GLOBAL | LightMap | m_FillLineWord | m_FillLineWord__8LightMapFPPUsPPUsiUi |
| 0x0045ebe0 | 96 | GLOBAL | LightMap | m_FillLineRGB | m_FillLineRGB__8LightMapFPPUcPPUciUi |
| 0x0045ec40 | 120 | GLOBAL | LightMap | m_FillLineRGB | m_FillLineRGB__8LightMapFPPUcPPUcPPUcPPUciUi |
| 0x0045ecc0 | 100 | GLOBAL | LightMap | m_Resize | m_Resize__8LightMapFP14RECTANGLE_INFOiiPPUc |
| 0x0045ed30 | 7436 | GLOBAL | LightMap | m_CreateLM | m_CreateLM__8LightMapFPUsP14RECTANGLE_INFOiPUiP14I3D_lit_object |
| 0x00460a40 | 100 | 13 | C_list<12TEXBL_STRUCT> | {dtor} | __dt__22C_list<12TEXBL_STRUCT>Fv |
| 0x00460ab0 | 96 | GLOBAL |  | LoadProgress__Ff | LoadProgress__Ff |
| 0x00460b10 | 1144 | GLOBAL |  | LoadHierarchy__FPCcUiPF21LS3D_CALLBACK_MESSAGEUiUiPv_vPvP9I3D_frame | LoadHierarchy__FPCcUiPF21LS3D_CALLBACK_MESSAGEUiUiPv_vPvP9I3D_frame |
| 0x00460f90 | 480 | GLOBAL |  | LoadAnimation__FPCcUiP17I3D_animation_set | LoadAnimation__FPCcUiP17I3D_animation_set |
| 0x00461170 | 1488 | GLOBAL |  | LoadImage__FPCcPC9_FILETIME | LoadImage__FPCcPC9_FILETIME |
| 0x00461740 | 1108 | GLOBAL |  | LoadMeshImage__Fv | LoadMeshImage__Fv |
| 0x00461ba0 | 388 | GLOBAL |  | LoadAnimation5ds__FPCcPPvPC9_FILETIME | LoadAnimation5ds__FPCcPPvPC9_FILETIME |
| 0x00461d30 | 80 | GLOBAL |  | sort_0_b__FP13sort_entry_32P13sort_entry_32PUiUi | sort_0_b__FP13sort_entry_32P13sort_entry_32PUiUi |
| 0x00461d80 | 88 | GLOBAL |  | sort_1_b__FP13sort_entry_32P13sort_entry_32PUiUi | sort_1_b__FP13sort_entry_32P13sort_entry_32PUiUi |
| 0x00461de0 | 84 | GLOBAL |  | sort_2_b__FP13sort_entry_32P13sort_entry_32PUiUi | sort_2_b__FP13sort_entry_32P13sort_entry_32PUiUi |
| 0x00461e40 | 92 | GLOBAL |  | sort_3_b__FP13sort_entry_32P13sort_entry_32PUiUi | sort_3_b__FP13sort_entry_32P13sort_entry_32PUiUi |
| 0x00461ea0 | 888 | GLOBAL |  | RadixSort32__FP13sort_entry_32i | RadixSort32__FP13sort_entry_32i |
| 0x00462220 | 328 | GLOBAL |  | GetObjectLOD__FP10I3D_objectRCf | GetObjectLOD__FP10I3D_objectRCf |
| 0x00462370 | 584 | LOCAL |  | ExpandVisualBB__FP10I3D_visualRA8_8S_vector | ExpandVisualBB__FP10I3D_visualRA8_8S_vector |
| 0x004625c0 | 4204 | LOCAL |  | AlphaBoxCmpFunc__FPCvPCv | AlphaBoxCmpFunc__FPCvPCv |
| 0x00463630 | 40 | GLOBAL | I3D_driver | initRender | initRender__10I3D_driverFv |
| 0x00463660 | 52 | GLOBAL | I3D_driver | closeRender | closeRender__10I3D_driverFv |
| 0x004636a0 | 9748 | GLOBAL | I3D_driver | Render | Render__10I3D_driverFP9I3D_sceneb |
| 0x00465cc0 | 48 | 13 | S_AlphaSortElement | {ctor} | __ct__18S_AlphaSortElementFv |
| 0x00465cf0 | 424 | LOCAL |  | VisualProc__FP10I3D_visual | VisualProc__FP10I3D_visual |
| 0x00465ea0 | 192 | 13 | C_list<P8I3D_area> | push_back | push_back__18C_list<P8I3D_area>FRCP8I3D_area |
| 0x00465f60 | 424 | GLOBAL |  | NextFFrameProc__FP9I3D_frame | NextFFrameProc__FP9I3D_frame |
| 0x00466110 | 808 | GLOBAL |  | NextFFrameProc2__FP9I3D_frame | NextFFrameProc2__FP9I3D_frame |
| 0x00466440 | 384 | GLOBAL | I3D_driver | BuildAllVBuffers | BuildAllVBuffers__10I3D_driverFP9I3D_scene |
| 0x004665c0 | 5072 | 13 | I3D_driver | m_DrawAllSortedFG | m_DrawAllSortedFG__10I3D_driverFv |
| 0x00467990 | 140 | 13 | ADD_FG_HELP_STRUCT | {dtor} | __dt__18ADD_FG_HELP_STRUCTFv |
| 0x00467a20 | 376 | GLOBAL |  | ComputeAndSetMaterial__FP13_LSIMATERIAL8ff | ComputeAndSetMaterial__FP13_LSIMATERIAL8ff |
| 0x00467ba0 | 768 | GLOBAL |  | UpdateAmbientLight__FP10I3D_sector | UpdateAmbientLight__FP10I3D_sector |
| 0x00467ea0 | 236 | GLOBAL |  | AllocLight__FP9I3D_light | AllocLight__FP9I3D_light |
| 0x00467f90 | 368 | GLOBAL |  | FreeLight__FP9I3D_light | FreeLight__FP9I3D_light |
| 0x00468100 | 328 | GLOBAL |  | SetLights__FUiiPUs | SetLights__FUiiPUs |
| 0x00468250 | 8 | GLOBAL |  | LightingSetSector__FP10I3D_sector | LightingSetSector__FP10I3D_sector |
| 0x00468260 | 1220 | GLOBAL |  | PrepareLights__FP10I3D_objectUi | PrepareLights__FP10I3D_objectUi |
| 0x00468730 | 988 | GLOBAL |  | TexelCallback__FRC8S_vectorfP9I3D_framePvPv | TexelCallback__FRC8S_vectorfP9I3D_framePvPv |
| 0x00468b10 | 3004 | GLOBAL |  | VertexLightingProcess__FR8S_vectorRC8S_vectorRC8S_vectorP12I3D_materialP9I3D_sceneUiPbP9I3D_framei | VertexLightingProcess__FR8S_vectorRC8S_vectorRC8S_vectorP12I3D_materialP9I3D_sceneUiPbP9I3D_framei |
| 0x004696d0 | 504 | GLOBAL |  | EnvProcess__FPvUiPvUiUiRC8S_matrixfi | EnvProcess__FPvUiPvUiUiRC8S_matrixfi |
| 0x004698d0 | 12 | GLOBAL |  | TEV | TEV |
| 0x004698e0 | 8 | GLOBAL |  | GetIGraph | GetIGraph |
| 0x004698f0 | 8 | GLOBAL |  | ISndGetDriver | ISndGetDriver |
| 0x00469900 | 8 | GLOBAL |  | I3DGetDriver | I3DGetDriver |
| 0x00469910 | 628 | GLOBAL |  | I3DGetResultMessage | I3DGetResultMessage |
| 0x00469b90 | 8 | GLOBAL |  | dbgClear__Fv | dbgClear__Fv |
| 0x00469ba0 | 44 | GLOBAL |  | dbgPrintf | dbgPrintf |
| 0x00469bd0 | 8 | GLOBAL |  | dbgPrintGuid__FPCv | dbgPrintGuid__FPCv |
| 0x00469be0 | 264 | GLOBAL |  | BBoxVertices__FRC8I3D_bbox | BBoxVertices__FRC8I3D_bbox |
| 0x00469cf0 | 88 | GLOBAL |  | ErrorHandler__F11LS3D_RESULT10E_SEVERITYPCcUi | ErrorHandler__F11LS3D_RESULT10E_SEVERITYPCcUi |
| 0x00469d50 | 120 | 13 | BreBre | {dtor} | __dt__6BreBreFv |
| 0x00469dd0 | 976 | GLOBAL | IGraph | {ctor} | __ct__6IGraphFv |
| 0x0046a1a0 | 124 | GLOBAL | IGraph | {dtor} | __dt__6IGraphFv |
| 0x0046a220 | 56 | GLOBAL | IGraph | CreateIShow | CreateIShow__6IGraphFv |
| 0x0046a260 | 96 | GLOBAL | IGraph | CreateITexture | CreateITexture__6IGraphFv |
| 0x0046a2c0 | 692 | GLOBAL | IGraph | CreateITexture | CreateITexture__6IGraphFPCcPCcUiPP8ITexture |
| 0x0046a580 | 104 | GLOBAL | IGraph | SetTexture | SetTexture__6IGraphFP8ITexturei |
| 0x0046a5f0 | 8 | GLOBAL | IGraph | GetMainHWND | GetMainHWND__6IGraphFv |
| 0x0046a600 | 8 | GLOBAL | IGraph | GetChildHWND | GetChildHWND__6IGraphFv |
| 0x0046a610 | 120 | GLOBAL | IGraph | SetAppName | SetAppName__6IGraphFPCc |
| 0x0046a690 | 132 | GLOBAL | IGraph | SetAppNameDebug | SetAppNameDebug__6IGraphFPCce |
| 0x0046a720 | 12 | GLOBAL | IGraph | GetAppName | GetAppName__6IGraphFv |
| 0x0046a730 | 240 | GLOBAL | IGraph | AddDlgHWND | AddDlgHWND__6IGraphFPv |
| 0x0046a820 | 208 | GLOBAL | IGraph | DeleteDlgHWND | DeleteDlgHWND__6IGraphFPv |
| 0x0046a8f0 | 8 | GLOBAL | IGraph | ProcessWinMessages | ProcessWinMessages__6IGraphFv |
| 0x0046a900 | 88 | GLOBAL | IGraph | Clear | Clear__6IGraphFUifUi |
| 0x0046a960 | 1524 | GLOBAL | IGraph | ResetRenderProps | ResetRenderProps__6IGraphFv |
| 0x0046af60 | 1928 | GLOBAL | IGraph | SetState | SetState__6IGraphF8DX_STATEUi |
| 0x0046b6f0 | 256 | GLOBAL | IGraph | GetState | GetState__6IGraphF8DX_STATE |
| 0x0046b7f0 | 28 | GLOBAL | IGraph | SetWorldMatrix | SetWorldMatrix__6IGraphFRC8S_matrix |
| 0x0046b810 | 60 | GLOBAL | IGraph | BeginScene | BeginScene__6IGraphFv |
| 0x0046b850 | 56 | GLOBAL | IGraph | EndScene | EndScene__6IGraphFv |
| 0x0046b890 | 428 | GLOBAL | IGraph | Present | Present__6IGraphFv |
| 0x0046ba40 | 1216 | GLOBAL | IGraph | DrawPrimitiveList | DrawPrimitiveList__6IGraphF14PRIMITIVE_TYPEUiPCv16LS3D_STREAM_TYPE |
| 0x0046bf00 | 1260 | GLOBAL | IGraph | DrawIndexedPrimitiveList | DrawIndexedPrimitiveList__6IGraphF14PRIMITIVE_TYPEUiPCvUiPCv16LS3D_STREAM_TYPE |
| 0x0046c3f0 | 868 | GLOBAL | IGraph | GetFormatName | GetFormatName__6IGraphCF10_LSIFORMAT |
| 0x0046c760 | 128 | LOCAL |  | SortModesCallback__FPCvPCv | SortModesCallback__FPCvPCv |
| 0x0046c7e0 | 164 | GLOBAL |  | IsDepthFormatOk__F10_LSIFORMAT10_LSIFORMAT10_LSIFORMAT | IsDepthFormatOk__F10_LSIFORMAT10_LSIFORMAT10_LSIFORMAT |
| 0x0046c890 | 72 | GLOBAL |  | IsTextureFormatOk__F10_LSIFORMAT10_LSIFORMAT | IsTextureFormatOk__F10_LSIFORMAT10_LSIFORMAT |
| 0x0046c8e0 | 76 | GLOBAL |  | ResetDevice__Fb | ResetDevice__Fb |
| 0x0046c930 | 84 | GLOBAL | IGraph | SetViewPort | SetViewPort__6IGraphFP13LS3D_VIEWPORT |
| 0x0046c990 | 12 | GLOBAL | IGraph | GetViewPort | GetViewPort__6IGraphFv |
| 0x0046c9a0 | 16 | GLOBAL | IGraph | SetInitFlags | SetInitFlags__6IGraphFUi |
| 0x0046c9b0 | 6232 | GLOBAL | IGraph | Init | Init__6IGraphFP16IGRAPH_INIT_DESC |
| 0x0046e210 | 8 | GLOBAL | IGraph | IsInit | IsInit__6IGraphFv |
| 0x0046e220 | 12 | GLOBAL | IGraph | GetResInitDesc | GetResInitDesc__6IGraphFv |
| 0x0046e230 | 996 | GLOBAL | IGraph | Close | Close__6IGraphFv |
| 0x0046e620 | 48 | GLOBAL | IGraph | Scrn_sx | Scrn_sx__6IGraphFv |
| 0x0046e650 | 48 | GLOBAL | IGraph | Scrn_sy | Scrn_sy__6IGraphFv |
| 0x0046e680 | 12 | GLOBAL | IGraph | BPP | BPP__6IGraphFv |
| 0x0046e690 | 12 | GLOBAL | IGraph | GetD3DFormat | GetD3DFormat__6IGraphFv |
| 0x0046e6a0 | 816 | GLOBAL | IGraph | GetImageInfo | GetImageInfo__6IGraphFPCcRUsRUsRUcUiPi |
| 0x0046e9d0 | 8 | GLOBAL | IGraph | SaveShot | SaveShot__6IGraphFPCcP5_RECTUiPv |
| 0x0046e9e0 | 456 | GLOBAL | IGraph | KeyboardInit | KeyboardInit__6IGraphFUi |
| 0x0046ebb0 | 8 | GLOBAL | IGraph | IsKeyboardInit | IsKeyboardInit__6IGraphFv |
| 0x0046ebc0 | 76 | GLOBAL | IGraph | KeyboardClose | KeyboardClose__6IGraphFv |
| 0x0046ec10 | 332 | GLOBAL | IGraph | UpdateKeyboardData | UpdateKeyboardData__6IGraphFv |
| 0x0046ed60 | 60 | GLOBAL | IGraph | ReadCharBuffer | ReadCharBuffer__6IGraphFv |
| 0x0046eda0 | 20 | GLOBAL | IGraph | ClearCharBuffer | ClearCharBuffer__6IGraphFv |
| 0x0046edc0 | 444 | GLOBAL | IGraph | ReadKey | ReadKey__6IGraphFv |
| 0x0046ef80 | 12 | GLOBAL | IGraph | ReadChar | ReadChar__6IGraphFv |
| 0x0046ef90 | 36 | GLOBAL | IGraph | TestKey | TestKey__6IGraphFUc |
| 0x0046efc0 | 112 | GLOBAL | IGraph | DefineKeys | DefineKeys__6IGraphFPCcUi |
| 0x0046f030 | 100 | GLOBAL | IGraph | ReadKeys | ReadKeys__6IGraphFv |
| 0x0046f0a0 | 52 | GLOBAL | IGraph | KeyShift | KeyShift__6IGraphFv |
| 0x0046f0e0 | 52 | GLOBAL | IGraph | KeyCtrl | KeyCtrl__6IGraphFv |
| 0x0046f120 | 52 | GLOBAL | IGraph | KeyAlt | KeyAlt__6IGraphFv |
| 0x0046f160 | 272 | GLOBAL | IGraph | GetShiftKeys | GetShiftKeys__6IGraphFb |
| 0x0046f270 | 92 | GLOBAL | IGraph | TranslateKey | TranslateKey__6IGraphFUcUi |
| 0x0046f2d0 | 8 | GLOBAL | IGraph | GetKeyName | GetKeyName__6IGraphFi |
| 0x0046f2e0 | 12 | GLOBAL | IGraph | GetRawInput | GetRawInput__6IGraphFv |
| 0x0046f2f0 | 8 | GLOBAL | IGraph | SetControlRumble | SetControlRumble__6IGraphFff |
| 0x0046f300 | 556 | GLOBAL | IGraph | MouseInit | MouseInit__6IGraphFUi |
| 0x0046f530 | 8 | GLOBAL | IGraph | IsMouseInit | IsMouseInit__6IGraphFv |
| 0x0046f540 | 96 | GLOBAL | IGraph | MouseClose | MouseClose__6IGraphFv |
| 0x0046f5a0 | 568 | GLOBAL | IGraph | UpdateMouseData | UpdateMouseData__6IGraphFv |
| 0x0046f7e0 | 12 | GLOBAL | IGraph | Mouse_rx | Mouse_rx__6IGraphFv |
| 0x0046f7f0 | 12 | GLOBAL | IGraph | Mouse_ry | Mouse_ry__6IGraphFv |
| 0x0046f800 | 12 | GLOBAL | IGraph | Mouse_rz | Mouse_rz__6IGraphFv |
| 0x0046f810 | 12 | GLOBAL | IGraph | SetMousePos | SetMousePos__6IGraphFii |
| 0x0046f820 | 8 | GLOBAL | IGraph | Mouse_x | Mouse_x__6IGraphFv |
| 0x0046f830 | 8 | GLOBAL | IGraph | Mouse_y | Mouse_y__6IGraphFv |
| 0x0046f840 | 120 | GLOBAL | IGraph | GetMouseButtons | GetMouseButtons__6IGraphFv |
| 0x0046f8c0 | 32 | GLOBAL | IGraph | NullTimer | NullTimer__6IGraphFv |
| 0x0046f8e0 | 140 | GLOBAL | IGraph | GetTimer | GetTimer__6IGraphFUiUi |
| 0x0046f970 | 552 | GLOBAL |  | m_AddNameToList__FP10C_list<Pc>PCcb | m_AddNameToList__FP10C_list<Pc>PCcb |
| 0x0046fba0 | 180 | GLOBAL |  | m_DeleteNameList__FP10C_list<Pc>PCc | m_DeleteNameList__FP10C_list<Pc>PCc |
| 0x0046fc60 | 104 | GLOBAL |  | m_ClearNameList__FP10C_list<Pc> | m_ClearNameList__FP10C_list<Pc> |
| 0x0046fcd0 | 12 | GLOBAL | IGraph | AddMapsDir | AddMapsDir__6IGraphFPCcb |
| 0x0046fce0 | 12 | GLOBAL | IGraph | DeleteMapsDir | DeleteMapsDir__6IGraphFPCc |
| 0x0046fcf0 | 12 | GLOBAL | IGraph | ClearMapDirs | ClearMapDirs__6IGraphFv |
| 0x0046fd00 | 12 | GLOBAL | IGraph | GetCachedTextures | GetCachedTextures__6IGraphFv |
| 0x0046fd10 | 12 | GLOBAL | IGraph | GetCachedTextureCnt | GetCachedTextureCnt__6IGraphFv |
| 0x0046fd20 | 12 | GLOBAL | IGraph | GetUncachedTextures | GetUncachedTextures__6IGraphFv |
| 0x0046fd30 | 12 | GLOBAL | IGraph | GetUncachedTextureCnt | GetUncachedTextureCnt__6IGraphFv |
| 0x0046fd40 | 84 | GLOBAL | IGraph | SetGammaRamp | SetGammaRamp__6IGraphFP13LS3DGAMMARAMPUi |
| 0x0046fda0 | 204 | GLOBAL | IGraph | GetGammaRamp | GetGammaRamp__6IGraphFP13LS3DGAMMARAMP |
| 0x0046fe70 | 3784 | GLOBAL | IGraph | ResizeImage | ResizeImage__6IGraphFiiUiPUciiUiPPUcUii |
| 0x00470d40 | 92 | 13 | C_list<P5IShow> | {dtor} | __dt__15C_list<P5IShow>Fv |
| 0x00470da0 | 92 | 13 | C_list<Pc> | {dtor} | __dt__10C_list<Pc>Fv |
| 0x00470e00 | 92 | 13 | C_list<10_LSIFORMAT> | {dtor} | __dt__20C_list<10_LSIFORMAT>Fv |
| 0x00470e60 | 92 | 13 | C_list<15_LSIDISPLAYMODE> | {dtor} | __dt__25C_list<15_LSIDISPLAYMODE>Fv |
| 0x00470ec0 | 92 | 13 | C_list<Pv> | {dtor} | __dt__10C_list<Pv>Fv |
| 0x00470f20 | 380 | GLOBAL | IShow | {ctor} | __ct__5IShowFv |
| 0x004710a0 | 300 | GLOBAL | IShow | {dtor} | __dt__5IShowFv |
| 0x004711d0 | 16 | GLOBAL | IShow | AddRef | AddRef__5IShowFv |
| 0x004711e0 | 116 | GLOBAL | IShow | Release | Release__5IShowFv |
| 0x00471260 | 768 | GLOBAL | IShow | Open | Open__5IShowFPCcUi |
| 0x00471560 | 8 | GLOBAL | IShow | IsInit | IsInit__5IShowFv |
| 0x00471570 | 344 | GLOBAL | IShow | Close | Close__5IShowFv |
| 0x004716d0 | 168 | GLOBAL | IShow | Play | Play__5IShowFv |
| 0x00471780 | 160 | GLOBAL | IShow | Stop | Stop__5IShowFv |
| 0x00471820 | 124 | GLOBAL | IShow | Pause | Pause__5IShowFv |
| 0x004718a0 | 8 | GLOBAL | IShow | m_PrepareVerts | m_PrepareVerts__5IShowFv |
| 0x004718b0 | 116 | GLOBAL | IShow | SetRect | SetRect__5IShowFP5_RECT |
| 0x00471930 | 64 | GLOBAL | IShow | GetRect | GetRect__5IShowFP5_RECT |
| 0x00471970 | 48 | GLOBAL | IShow | GetVideoRect | GetVideoRect__5IShowFP5_RECT |
| 0x004719a0 | 20 | GLOBAL | IShow | GetEvent | GetEvent__5IShowFv |
| 0x004719c0 | 124 | GLOBAL | IShow | SetCurrentPos | SetCurrentPos__5IShowFUi |
| 0x00471a40 | 164 | GLOBAL | IShow | GetCurrentPos | GetCurrentPos__5IShowFPUi |
| 0x00471af0 | 240 | GLOBAL | IShow | SetVolume | SetVolume__5IShowFf |
| 0x00471be0 | 8 | GLOBAL | IShow | GetRegionCnt | GetRegionCnt__5IShowFv |
| 0x00471bf0 | 44 | GLOBAL | IShow | GetRegions | GetRegions__5IShowFv |
| 0x00471c20 | 8 | GLOBAL | IShow | ProcessBinkFrame | ProcessBinkFrame__5IShowFv |
| 0x00471c30 | 8 | GLOBAL | IShow | RenderCurrentFrame | RenderCurrentFrame__5IShowFv |
| 0x00471c40 | 296 | GLOBAL | IShow | RenderVideoUntilKeyPressOrEnd | RenderVideoUntilKeyPressOrEnd__5IShowFv |
| 0x00471d70 | 8 | 13 | IShow | GetVolume | GetVolume__5IShowFv |
| 0x00471d80 | 8 | 13 | IShow | GetDecodedFrameNum | GetDecodedFrameNum__5IShowFv |
| 0x00471d90 | 56 | GLOBAL | ITexture | {ctor} | __ct__8ITextureFv |
| 0x00471dd0 | 84 | GLOBAL | ITexture | {dtor} | __dt__8ITextureFv |
| 0x00471e30 | 92 | GLOBAL | ITexture | Release | Release__8ITextureFv |
| 0x00471e90 | 12524 | GLOBAL | ITexture | Create | Create__8ITextureFPCcPCcUiUsUsUs |
| 0x00474f80 | 808 | GLOBAL | ITexture | m_TryPresavedDXT | m_TryPresavedDXT__8ITextureFPCcPCciP9_FILETIMEP9_FILETIME |
| 0x004752b0 | 8 | GLOBAL | ITexture | m_TryPresaved_555_565 | m_TryPresaved_555_565__8ITextureFPCcP9_FILETIME |
| 0x004752c0 | 508 | GLOBAL | ITexture | Close | Close__8ITextureFv |
| 0x004754c0 | 140 | GLOBAL | ITexture | Lock | Lock__8ITextureFP5_RECTRUiUs |
| 0x00475550 | 84 | GLOBAL | ITexture | Unlock | Unlock__8ITextureFv |
| 0x004755b0 | 8 | GLOBAL | ITexture | Preload | Preload__8ITextureFv |
| 0x004755c0 | 716 | GLOBAL | ITexture | Fill | Fill__8ITextureFP5_RECTUiUs |
| 0x00475890 | 2620 | GLOBAL |  | LoadImage__FPCcUiUiUiPPvPPvP19tagBITMAPINFOHEADERbP9_FILETIMEi | LoadImage__FPCcUiUiUiPPvPPvP19tagBITMAPINFOHEADERbP9_FILETIMEi |
| 0x004762d0 | 4012 | GLOBAL | ITexture | m_Convert | m_Convert__8ITextureFPUcP11ILSISurfaceUiUii |
| 0x00477280 | 8 | 13 | S_vector4 | {ctor} | __ct__9S_vector4Fv |
| 0x00477290 | 312 | GLOBAL |  | CompressTexture__FPCciP11ILSISurfaceP11ILSISurface | CompressTexture__FPCciP11ILSISurfaceP11ILSISurface |
| 0x004773d0 | 96 | GLOBAL | ISND_3Dlistener | {ctor} | __ct__15ISND_3DlistenerFv |
| 0x00477430 | 76 | GLOBAL | ISND_3Dlistener | {dtor} | __dt__15ISND_3DlistenerFv |
| 0x00477480 | 16 | GLOBAL | ISND_3Dlistener | AddRef | AddRef__15ISND_3DlistenerFv |
| 0x00477490 | 92 | GLOBAL | ISND_3Dlistener | Release | Release__15ISND_3DlistenerFv |
| 0x004774f0 | 32 | GLOBAL | ISND_3Dlistener | SetPos | SetPos__15ISND_3DlistenerFRC8S_vector |
| 0x00477510 | 56 | GLOBAL | ISND_3Dlistener | SetDir | SetDir__15ISND_3DlistenerFRC8S_vectorRC8S_vector |
| 0x00477550 | 32 | GLOBAL | ISND_3Dlistener | SetVelocity | SetVelocity__15ISND_3DlistenerFRC8S_vector |
| 0x00477570 | 12 | GLOBAL | ISND_3Dlistener | SetDistanceFactor | SetDistanceFactor__15ISND_3DlistenerFf |
| 0x00477580 | 12 | GLOBAL | ISND_3Dlistener | SetDopplerFactor | SetDopplerFactor__15ISND_3DlistenerFf |
| 0x00477590 | 12 | GLOBAL | ISND_3Dlistener | SetRolloffFactor | SetRolloffFactor__15ISND_3DlistenerFf |
| 0x004775a0 | 8 | GLOBAL | ISND_3Dlistener | Update | Update__15ISND_3DlistenerFv |
| 0x004775b0 | 8 | GLOBAL | ISND_3Dlistener | SetEAXProperty | SetEAXProperty__15ISND_3DlistenerFiUi |
| 0x004775c0 | 8 | GLOBAL | ISND_3Dlistener | SetEAX3Property | SetEAX3Property__15ISND_3DlistenerFUiPvUi |
| 0x004775d0 | 8 | GLOBAL | ISND_3Dlistener | GetEAX3Property | GetEAX3Property__15ISND_3DlistenerFUiPvUiPUi |
| 0x004775e0 | 8 | GLOBAL | ISND_3Dlistener | CommitDeferredSettings | CommitDeferredSettings__15ISND_3DlistenerFv |
| 0x004775f0 | 60 | GLOBAL | ISND_3Dsound | {ctor} | __ct__12ISND_3DsoundFv |
| 0x00477630 | 116 | GLOBAL | ISND_3Dsound | {dtor} | __dt__12ISND_3DsoundFv |
| 0x004776b0 | 8 | GLOBAL | ISND_3Dsound | Is3DInit | Is3DInit__12ISND_3DsoundCFv |
| 0x004776c0 | 8 | GLOBAL | ISND_3Dsound | SetPos | SetPos__12ISND_3DsoundFRC8S_vector |
| 0x004776d0 | 8 | GLOBAL | ISND_3Dsound | SetDir | SetDir__12ISND_3DsoundFRC8S_vector |
| 0x004776e0 | 8 | GLOBAL | ISND_3Dsound | SetVelocity | SetVelocity__12ISND_3DsoundFRC8S_vector |
| 0x004776f0 | 8 | GLOBAL | ISND_3Dsound | SetDistances | SetDistances__12ISND_3DsoundFff |
| 0x00477700 | 8 | GLOBAL | ISND_3Dsound | SetCone | SetCone__12ISND_3DsoundFff |
| 0x00477710 | 8 | GLOBAL | ISND_3Dsound | SetOutVolume | SetOutVolume__12ISND_3DsoundFf |
| 0x00477720 | 108 | GLOBAL | ISND_3Dsound | Open | Open__12ISND_3DsoundFP11ISND_sourceUi |
| 0x00477790 | 80 | GLOBAL | ISND_3Dsound | Close | Close__12ISND_3DsoundFv |
| 0x004777e0 | 8 | 13 | ISND_sound | GetSource | GetSource__10ISND_soundFv |
| 0x004777f0 | 68 | GLOBAL | ISND_driver | {ctor} | __ct__11ISND_driverFv |
| 0x00477840 | 144 | GLOBAL | ISND_driver | {dtor} | __dt__11ISND_driverFv |
| 0x004778d0 | 8 | GLOBAL | ISND_driver | IsInit | IsInit__11ISND_driverCFv |
| 0x004778e0 | 200 | GLOBAL | ISND_driver | Close | Close__11ISND_driverFv |
| 0x004779b0 | 8 | GLOBAL | ISND_driver | InitBuffer | InitBuffer__11ISND_driverFiii |
| 0x004779c0 | 352 | GLOBAL | ISND_driver | SetVolume | SetVolume__11ISND_driverFf |
| 0x00477b20 | 8 | GLOBAL | ISND_driver | GetCaps | GetCaps__11ISND_driverCFPUi |
| 0x00477b30 | 44 | GLOBAL | ISND_driver | CreateSource | CreateSource__11ISND_driverFv |
| 0x00477b60 | 88 | GLOBAL | ISND_driver | CreateSource | CreateSource__11ISND_driverFPP11ISND_source |
| 0x00477bc0 | 44 | GLOBAL | ISND_driver | CreateSound | CreateSound__11ISND_driverFv |
| 0x00477bf0 | 88 | GLOBAL | ISND_driver | CreateSound | CreateSound__11ISND_driverFPP10ISND_sound |
| 0x00477c50 | 44 | GLOBAL | ISND_driver | Create3DSound | Create3DSound__11ISND_driverFv |
| 0x00477c80 | 88 | GLOBAL | ISND_driver | Create3DSound | Create3DSound__11ISND_driverFPP12ISND_3Dsound |
| 0x00477ce0 | 56 | GLOBAL | ISND_driver | CreateListener | CreateListener__11ISND_driverFv |
| 0x00477d20 | 88 | GLOBAL | ISND_driver | CreateListener | CreateListener__11ISND_driverFPP15ISND_3Dlistener |
| 0x00477d80 | 176 | GLOBAL | ISND_driver | InitDSound | InitDSound__11ISND_driverFPviiiUi |
| 0x00477e30 | 92 | GLOBAL | ISND_driver | Init | Init__11ISND_driverFPviiiUi |
| 0x00477e90 | 76 | GLOBAL | ISND_driver | Init | Init__11ISND_driverFP9ISND_INIT |
| 0x00477ee0 | 1560 | GLOBAL |  | UpdateAllTerminatedBuffs__Fv | UpdateAllTerminatedBuffs__Fv |
| 0x00478500 | 48 | GLOBAL | ISND_sound | {ctor} | __ct__10ISND_soundFv |
| 0x00478530 | 104 | GLOBAL | ISND_sound | {dtor} | __dt__10ISND_soundFv |
| 0x004785a0 | 108 | GLOBAL | ISND_sound | Release | Release__10ISND_soundFv |
| 0x00478610 | 180 | GLOBAL | ISND_sound | Open | Open__10ISND_soundFP11ISND_sourceUi |
| 0x004786d0 | 104 | GLOBAL | ISND_sound | Close | Close__10ISND_soundFv |
| 0x00478740 | 96 | GLOBAL | ISND_sound | Play | Play__10ISND_soundFP11ISND_sourceUi |
| 0x004787a0 | 136 | GLOBAL | ISND_sound | Play | Play__10ISND_soundFUiUi |
| 0x00478830 | 60 | GLOBAL | ISND_sound | Stop | Stop__10ISND_soundFv |
| 0x00478870 | 56 | GLOBAL | ISND_sound | GetCurrTime | GetCurrTime__10ISND_soundCFv |
| 0x004788b0 | 60 | GLOBAL | ISND_sound | SetCurrTime | SetCurrTime__10ISND_soundFUi |
| 0x004788f0 | 180 | GLOBAL | ISND_sound | SetVolume | SetVolume__10ISND_soundFf |
| 0x004789b0 | 128 | GLOBAL | ISND_sound | SetVolumeLOG | SetVolumeLOG__10ISND_soundFf |
| 0x00478a30 | 184 | GLOBAL | ISND_sound | SetVolumeLOG | SetVolumeLOG__10ISND_soundFi |
| 0x00478af0 | 356 | GLOBAL | ISND_sound | SetFrequency | SetFrequency__10ISND_soundFf |
| 0x00478c60 | 76 | GLOBAL | ISND_sound | SetPanning | SetPanning__10ISND_soundFf |
| 0x00478cb0 | 64 | GLOBAL | ISND_sound | IsPlaying | IsPlaying__10ISND_soundCFv |
| 0x00478cf0 | 20 | GLOBAL | ISND_sound | LSI_Lock | LSI_Lock__10ISND_soundFi |
| 0x00478d10 | 92 | 13 | C_list<25TERMINATED_CHANNEL_STRUCT> | {dtor} | __dt__35C_list<25TERMINATED_CHANNEL_STRUCT>Fv |
| 0x00478d70 | 92 | 13 | C_list<14CHANNEL_STRUCT> | {dtor} | __dt__24C_list<14CHANNEL_STRUCT>Fv |
| 0x00478dd0 | 64 | GLOBAL | ISND_source | {ctor} | __ct__11ISND_sourceFv |
| 0x00478e10 | 284 | GLOBAL | ISND_source | {dtor} | __dt__11ISND_sourceFv |
| 0x00478f30 | 92 | GLOBAL | ISND_source | Release | Release__11ISND_sourceFv |
| 0x00478f90 | 488 | GLOBAL | ISND_source | Open | Open__11ISND_sourceFPCcUi |
| 0x00479180 | 220 | GLOBAL | ISND_source | Close | Close__11ISND_sourceFv |
| 0x00479260 | 156 | GLOBAL | ISND_source | GetPlayTime | GetPlayTime__11ISND_sourceCFv |
| 0x00479300 | 44 | GLOBAL | ISND_source | GetFormat | GetFormat__11ISND_sourceFP13S_wave_format |
| 0x00479330 | 36 | GLOBAL | C_VBuffer | {ctor} | __ct__9C_VBufferFv |
| 0x00479360 | 312 | GLOBAL | C_VBuffer | {dtor} | __dt__9C_VBufferFv |
| 0x004794a0 | 116 | GLOBAL | C_VBuffer | m_Init | m_Init__9C_VBufferFUiUiUi13VERTEX_FORMAT |
| 0x00479520 | 236 | GLOBAL | C_VBManager | {ctor} | __ct__11C_VBManagerFv |
| 0x00479610 | 180 | GLOBAL | C_VBManager | {dtor} | __dt__11C_VBManagerFv |
| 0x004796d0 | 344 | GLOBAL | C_VBManager | m_Alloc | m_Alloc__11C_VBManagerFUiUiUiR18COMPLETE_VB_STRUCT13VERTEX_FORMATb |
| 0x00479830 | 52 | GLOBAL | C_VBManager | m_Free | m_Free__11C_VBManagerFR18COMPLETE_VB_STRUCT |
| 0x00479870 | 8 | GLOBAL | C_VBManager | m_UnlockAllVBuffers | m_UnlockAllVBuffers__11C_VBManagerFv |
| 0x00479880 | 48 | GLOBAL | C_VBManager | m_FreeAllUnusedVBuffers | m_FreeAllUnusedVBuffers__11C_VBManagerFP9C_VBuffer |
| 0x004798b0 | 96 | GLOBAL | C_VBManager | m_FreeAllVBuffers | m_FreeAllVBuffers__11C_VBManagerFv |
| 0x00479910 | 132 | GLOBAL | C_VBManager | m_FreeAllVRAMVBuffers | m_FreeAllVRAMVBuffers__11C_VBManagerFv |
| 0x004799a0 | 12 | GLOBAL | C_IBuffer | {ctor} | __ct__9C_IBufferFv |
| 0x004799b0 | 104 | GLOBAL | C_IBuffer | {dtor} | __dt__9C_IBufferFv |
| 0x00479a20 | 60 | GLOBAL | C_IBuffer | m_Init | m_Init__9C_IBufferFUiUi |
| 0x00479a60 | 8 | GLOBAL | C_IBManager | {ctor} | __ct__11C_IBManagerFv |
| 0x00479a70 | 68 | GLOBAL | C_IBManager | {dtor} | __dt__11C_IBManagerFv |
| 0x00479ac0 | 156 | GLOBAL | C_IBManager | m_Alloc | m_Alloc__11C_IBManagerFUiUiR18COMPLETE_IB_STRUCT |
| 0x00479b60 | 52 | GLOBAL | C_IBManager | m_Free | m_Free__11C_IBManagerFR18COMPLETE_IB_STRUCT |
| 0x00479ba0 | 8 | GLOBAL | C_IBManager | m_UnlockAllIBuffers | m_UnlockAllIBuffers__11C_IBManagerFv |
| 0x00479bb0 | 48 | GLOBAL | C_IBManager | m_FreeAllUnusedIBuffers | m_FreeAllUnusedIBuffers__11C_IBManagerFv |
| 0x00479be0 | 8 | GLOBAL | C_IBManager | m_FreeAllIBuffers | m_FreeAllIBuffers__11C_IBManagerFv |
| 0x00479bf0 | 48 | GLOBAL | C_IBManager | m_FreeAllVRAMIBuffers | m_FreeAllVRAMIBuffers__11C_IBManagerFv |
| 0x00479c20 | 80 | GLOBAL | C_vid | {ctor} | __ct__5C_vidFv |
| 0x00479c70 | 68 | GLOBAL | C_vid | {dtor} | __dt__5C_vidFv |
| 0x00479cc0 | 32 | GLOBAL | C_vid | Reset | Reset__5C_vidFv |
| 0x00479ce0 | 8 | GLOBAL | C_vid | IsSameAB | IsSameAB__5C_vidFv |
| 0x00479cf0 | 292 | GLOBAL | C_vid | ControlerIsOK | ControlerIsOK__5C_vidFv |
| 0x00479e20 | 4088 | GLOBAL | C_vid | Update | Update__5C_vidFUi |
| 0x0047ae20 | 296 | GLOBAL | C_vid | ReadKey | ReadKey__5C_vidFv |
| 0x0047af50 | 440 | GLOBAL | C_vid | TestKey | TestKey__5C_vidFi |
| 0x0047b110 | 316 | GLOBAL | C_vid | GetIcon | GetIcon__5C_vidFi |
| 0x0047b250 | 76 | GLOBAL | G_Inventory | GetWeaponInRightHand | GetWeaponInRightHand__11G_InventoryFv |
| 0x0047b2a0 | 8 | GLOBAL | G_Inventory | GetWeaponInLeftHand | GetWeaponInLeftHand__11G_InventoryFv |
| 0x0047b2b0 | 64 | GLOBAL | G_Inventory | Select | Select__11G_InventoryFUiUiPQ23gif22vector<10S_GameItem,8> |
| 0x0047b2f0 | 128 | GLOBAL | G_Inventory | SelectByID | SelectByID__11G_InventoryFUiPQ23gif22vector<10S_GameItem,8> |
| 0x0047b370 | 612 | GLOBAL | G_Inventory | Holster | Holster__11G_InventoryFPQ23gif22vector<10S_GameItem,8>bb |
| 0x0047b5e0 | 228 | GLOBAL | G_Inventory | HaveSomeWeapon | HaveSomeWeapon__11G_InventoryFbUi |
| 0x0047b6d0 | 228 | GLOBAL | G_Inventory | HaveItems | HaveItems__11G_InventoryFPQ23std52vector<10S_GameItem,Q23std23allocator<10S_GameItem>>Ui |
| 0x0047b7c0 | 572 | GLOBAL | G_Inventory | CyclePrepare | CyclePrepare__11G_InventoryFv |
| 0x0047ba00 | 936 | GLOBAL | G_Inventory | CycleUp | CycleUp__11G_InventoryFPQ23gif22vector<10S_GameItem,8> |
| 0x0047bdb0 | 908 | GLOBAL | G_Inventory | CycleDown | CycleDown__11G_InventoryFPQ23gif22vector<10S_GameItem,8> |
| 0x0047c140 | 148 | GLOBAL | G_Inventory | IsIn | IsIn__11G_InventoryFUib |
| 0x0047c1e0 | 8 | GLOBAL | G_Inventory | HaveSpaceFor | HaveSpaceFor__11G_InventoryFUib |
| 0x0047c1f0 | 188 | GLOBAL | G_Inventory | DobijNaboje | DobijNaboje__11G_InventoryFP10S_GameItemP10S_GameItem |
| 0x0047c2b0 | 2076 | GLOBAL | G_Inventory | InsertNew | InsertNew__11G_InventoryFP10S_GameItemPQ23gif22vector<10S_GameItem,8> |
| 0x0047cad0 | 1168 | GLOBAL | G_Inventory | Insert | Insert__11G_InventoryFP10S_GameItemb |
| 0x0047cf60 | 40 | GLOBAL | G_Inventory | RemoveInHand | RemoveInHand__11G_InventoryFv |
| 0x0047cf90 | 216 | GLOBAL | G_Inventory | RemoveByIdx | RemoveByIdx__11G_InventoryFUiUi |
| 0x0047d070 | 584 | GLOBAL | G_Inventory | Remove | Remove__11G_InventoryFP10S_GameItemPQ23gif22vector<10S_GameItem,8> |
| 0x0047d2c0 | 8 | GLOBAL | G_Inventory | SetCanHaveBigWeapon | SetCanHaveBigWeapon__11G_InventoryFbPQ23gif22vector<10S_GameItem,8> |
| 0x0047d2d0 | 8 | GLOBAL | G_Inventory | SetInCar | SetInCar__11G_InventoryFbPQ23gif22vector<10S_GameItem,8> |
| 0x0047d2e0 | 440 | GLOBAL | G_Inventory | EnemyToList | EnemyToList__11G_InventoryFPQ23gif23vector<P10S_GameItem,8> |
| 0x0047d4a0 | 256 | GLOBAL | G_Inventory | RemoveByPtr | RemoveByPtr__11G_InventoryFP10S_GameItem |
| 0x0047d5a0 | 116 | GLOBAL | G_Inventory | GetAmmo | GetAmmo__11G_InventoryFPi |
| 0x0047d620 | 164 | GLOBAL | G_Inventory | GetAmmoByID | GetAmmoByID__11G_InventoryFUiPi |
| 0x0047d6d0 | 84 | GLOBAL | G_Inventory | Vystrel | Vystrel__11G_InventoryFv |
| 0x0047d730 | 264 | GLOBAL | G_Inventory | Nabij | Nabij__11G_InventoryFv |
| 0x0047d840 | 148 | GLOBAL | G_Inventory | NabijJeden | NabijJeden__11G_InventoryFv |
| 0x0047d8e0 | 112 | GLOBAL | G_Inventory | MuzeNabitJeden | MuzeNabitJeden__11G_InventoryFv |
| 0x0047d950 | 8 | GLOBAL | G_Inventory | _Sesyp | _Sesyp__11G_InventoryFv |
| 0x0047d960 | 8 | GLOBAL | G_Inventory | SaveGameGetSize | SaveGameGetSize__11G_InventoryFv |
| 0x0047d970 | 216 | GLOBAL | G_Inventory | SaveGameSave | SaveGameSave__11G_InventoryFPPv |
| 0x0047da50 | 472 | GLOBAL | G_Inventory | SaveGameLoad | SaveGameLoad__11G_InventoryFPPv |
| 0x0047dc30 | 1264 | GLOBAL | G_Inventory | ItemPickUp | ItemPickUp__11G_InventoryFP11G_InventoryPQ23gif22vector<10S_GameItem,8>PQ23gif22vector<10S_GameItem,8>PQ23gif22vector<10S_GameItem,8>PQ23std56vector<P11G_Inventory,Q23std25allocator<P11G_Inventory>> |
| 0x0047e120 | 1244 | WEAK | std::__vector_imp<10S_GameItem,Q23std23allocator<10S_GameItem>,0> | insert | insert__Q23std60__vector_imp<10S_GameItem,Q23std23allocator<10S_GameItem>,0>FP10S_GameItemUiRC10S_GameItem |
| 0x0047e600 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std23allocator<10S_GameItem>,Ui,1> | swap | swap__Q310Metrowerks7details57compressed_pair_imp<Q23std23allocator<10S_GameItem>,Ui,1>FRQ310Metrowerks7details57compressed_pair_imp<Q23std23allocator<10S_GameItem>,Ui,1> |
| 0x0047e620 | 12 | GLOBAL | C_Particle | {ctor} | __ct__10C_ParticleFv |
| 0x0047e630 | 68 | GLOBAL | C_Particle | {dtor} | __dt__10C_ParticleFv |
| 0x0047e680 | 760 | GLOBAL | C_Particle | Create | Create__10C_ParticleFRC8S_matrixP12C_PData_Typeii |
| 0x0047e980 | 1352 | GLOBAL | C_Particle | UpdateDataByPhase | UpdateDataByPhase__10C_ParticleFRC8S_matrix |
| 0x0047eed0 | 776 | GLOBAL | C_Particle | Tick | Tick__10C_ParticleFi |
| 0x0047f1e0 | 168 | GLOBAL | C_ParticleEmitter | {ctor} | __ct__17C_ParticleEmitterFv |
| 0x0047f290 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14C_ParticleType>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14C_ParticleType>,Ui,1>FUi |
| 0x0047f2a0 | 364 | GLOBAL | C_ParticleEmitter | {dtor} | __dt__17C_ParticleEmitterFv |
| 0x0047f410 | 1072 | GLOBAL | C_ParticleEmitter | Init | Init__17C_ParticleEmitterFP9I3D_sceneP9I3D_frameRC8S_matrixP15C_PData_Emitterb |
| 0x0047f840 | 780 | GLOBAL | C_ParticleEmitter | ReInit | ReInit__17C_ParticleEmitterFP9I3D_sceneP9I3D_frameRC8S_matrixb |
| 0x0047fb50 | 108 | GLOBAL | C_ParticleEmitter | Destroy | Destroy__17C_ParticleEmitterFv |
| 0x0047fbc0 | 448 | GLOBAL | C_ParticleEmitter | CreateParticle | CreateParticle__17C_ParticleEmitterFP12C_PData_TypeRC8S_matrixii |
| 0x0047fd80 | 128 | 13 | C_Pool<10C_Particle> | Add | Add__20C_Pool<10C_Particle>FP10C_Particle |
| 0x0047fe00 | 240 | GLOBAL | C_ParticleEmitter | SpawnOnCreate | SpawnOnCreate__17C_ParticleEmitterFv |
| 0x0047fef0 | 676 | GLOBAL | C_ParticleEmitter | SpawnParticles | SpawnParticles__17C_ParticleEmitterFi |
| 0x004801a0 | 260 | GLOBAL | C_ParticleEmitter | DeleteDeathParticles | DeleteDeathParticles__17C_ParticleEmitterFv |
| 0x004802b0 | 196 | GLOBAL | C_ParticleEmitter | TickParticles | TickParticles__17C_ParticleEmitterFi |
| 0x00480380 | 1520 | GLOBAL | C_ParticleEmitter | SortParticlesAndCreateBBOX | SortParticlesAndCreateBBOX__17C_ParticleEmitterFb |
| 0x00480970 | 1308 | GLOBAL | C_ParticleEmitter | PrepareVBuffer | PrepareVBuffer__17C_ParticleEmitterFv |
| 0x00480e90 | 232 | GLOBAL | C_ParticleEmitter | Tick | Tick__17C_ParticleEmitterFi |
| 0x00480f80 | 136 | 13 | C_ParticleEmitter | GetWorldMatrix | GetWorldMatrix__17C_ParticleEmitterFv |
| 0x00481010 | 8 | GLOBAL | C_ParticleEmitter | UpdateOptimized | UpdateOptimized__17C_ParticleEmitterFi |
| 0x00481020 | 1364 | WEAK | std::__vector_imp<14C_ParticleType,Q23std27allocator<14C_ParticleType>,0> | insert | insert__Q23std68__vector_imp<14C_ParticleType,Q23std27allocator<14C_ParticleType>,0>FP14C_ParticleTypeUiRC14C_ParticleType |
| 0x00481580 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std27allocator<14C_ParticleType>,Ui,1> | swap | swap__Q310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14C_ParticleType>,Ui,1>FRQ310Metrowerks7details61compressed_pair_imp<Q23std27allocator<14C_ParticleType>,Ui,1> |
| 0x004815a0 | 80 | WEAK | std::__vector_deleter<14C_ParticleType,Q23std27allocator<14C_ParticleType>> | clear | clear__Q23std70__vector_deleter<14C_ParticleType,Q23std27allocator<14C_ParticleType>>Fv |
| 0x004815f0 | 104 | WEAK | std::__vector_deleter<14C_ParticleType,Q23std27allocator<14C_ParticleType>> | {dtor} | __dt__Q23std70__vector_deleter<14C_ParticleType,Q23std27allocator<14C_ParticleType>>Fv |
| 0x00481660 | 188 | 13 | C_Pool<10C_Particle> | {dtor} | __dt__20C_Pool<10C_Particle>Fv |
| 0x00481720 | 188 | GLOBAL | C_ParticleManager | {ctor} | __ct__17C_ParticleManagerFv |
| 0x004817e0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15C_ParticleInUse>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15C_ParticleInUse>,Ui,1>FUi |
| 0x004817f0 | 140 | 13 | std::vector<P17C_ParticleEmitter,Q23std31allocator<P17C_ParticleEmitter>> | {dtor} | __dt__Q23std68vector<P17C_ParticleEmitter,Q23std31allocator<P17C_ParticleEmitter>>Fv |
| 0x00481880 | 52 | 13 | std::vector<P17C_ParticleEmitter,Q23std31allocator<P17C_ParticleEmitter>> | {ctor} | __ct__Q23std68vector<P17C_ParticleEmitter,Q23std31allocator<P17C_ParticleEmitter>>Fv |
| 0x004818c0 | 228 | GLOBAL | C_ParticleManager | {dtor} | __dt__17C_ParticleManagerFv |
| 0x004819b0 | 184 | GLOBAL | C_ParticleManager | Mission_Start | Mission_Start__17C_ParticleManagerFv |
| 0x00481a70 | 260 | GLOBAL | C_ParticleManager | Mission_Done | Mission_Done__17C_ParticleManagerFv |
| 0x00481b80 | 1712 | GLOBAL | C_ParticleManager | LoadData | LoadData__17C_ParticleManagerFv |
| 0x00482230 | 128 | 13 | C_chunk | Size | Size__7C_chunkCFv |
| 0x004822b0 | 2504 | 13 | C_PData_Type | LoadData | LoadData__12C_PData_TypeFP7C_chunk |
| 0x00482c80 | 340 | 13 | C_PData_Type | {ctor} | __ct__12C_PData_TypeFv |
| 0x00482de0 | 20 | 13 | std::__vector_deleter<7UV_RECT,Q23std19allocator<7UV_RECT>> | {ctor} | __ct__Q23std54__vector_deleter<7UV_RECT,Q23std19allocator<7UV_RECT>>Fv |
| 0x00482e00 | 168 | 13 | C_chunk | RAscend | RAscend__7C_chunkFRUs |
| 0x00482eb0 | 76 | 13 | C_PData | {dtor} | __dt__7C_PDataFv |
| 0x00482f00 | 176 | GLOBAL | C_ParticleManager | CreateParticle | CreateParticle__17C_ParticleManagerFiRC8S_vectorRC8S_vectorP9I3D_framebb |
| 0x00482fb0 | 424 | GLOBAL | C_ParticleManager | CreateParticle | CreateParticle__17C_ParticleManagerFiRC8S_matrixP9I3D_framebb |
| 0x00483160 | 224 | GLOBAL | C_ParticleManager | ValidateParticleData | ValidateParticleData__17C_ParticleManagerFP15C_PData_Emitter |
| 0x00483240 | 292 | GLOBAL | C_ParticleManager | ClientTick | ClientTick__17C_ParticleManagerFi |
| 0x00483370 | 228 | GLOBAL | C_ParticleManager | FindGuid | FindGuid__17C_ParticleManagerFiRi |
| 0x00483460 | 84 | GLOBAL | C_ParticleManager | DestroyParticle | DestroyParticle__17C_ParticleManagerFi |
| 0x004834c0 | 52 | GLOBAL | C_ParticleManager | PauseParticle | PauseParticle__17C_ParticleManagerFib |
| 0x00483500 | 100 | GLOBAL | C_ParticleManager | ClearEmitters | ClearEmitters__17C_ParticleManagerFv |
| 0x00483570 | 120 | GLOBAL | C_ParticleManager | ClearData | ClearData__17C_ParticleManagerFv |
| 0x004835f0 | 300 | 13 | C_PData_Emitter | {dtor} | __dt__15C_PData_EmitterFv |
| 0x00483720 | 300 | 13 | C_PData_Type | {dtor} | __dt__12C_PData_TypeFv |
| 0x00483850 | 152 | 13 | C_PData_Phase | {dtor} | __dt__13C_PData_PhaseFv |
| 0x004838f0 | 32 | GLOBAL | C_ParticleManager | IsParticleAlive | IsParticleAlive__17C_ParticleManagerFi |
| 0x00483910 | 268 | GLOBAL | C_ParticleManager | CreateSParticle | CreateSParticle__17C_ParticleManagerFiRC8S_matrixP9I3D_frame |
| 0x00483a20 | 1100 | GLOBAL | C_ParticleManager | LoadStaticParticles | LoadStaticParticles__17C_ParticleManagerFv |
| 0x00483e70 | 1472 | GLOBAL | C_ParticleManager | LoadStaticParticles_100 | LoadStaticParticles_100__17C_ParticleManagerFR7C_chunk |
| 0x00484430 | 1384 | GLOBAL | C_ParticleManager | LoadStaticParticles_201 | LoadStaticParticles_201__17C_ParticleManagerFR7C_chunk |
| 0x004849a0 | 224 | GLOBAL | C_ParticleManager | PreloadTextures | PreloadTextures__17C_ParticleManagerFv |
| 0x00484a80 | 96 | GLOBAL | C_ParticleManager | ReleasePreloadedTextures | ReleasePreloadedTextures__17C_ParticleManagerFv |
| 0x00484ae0 | 1180 | WEAK | std::__vector_imp<7UV_RECT,Q23std19allocator<7UV_RECT>,0> | insert | insert__Q23std52__vector_imp<7UV_RECT,Q23std19allocator<7UV_RECT>,0>FP7UV_RECTUiRC7UV_RECT |
| 0x00484f80 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std19allocator<7UV_RECT>,Ui,1> | swap | swap__Q310Metrowerks7details53compressed_pair_imp<Q23std19allocator<7UV_RECT>,Ui,1>FRQ310Metrowerks7details53compressed_pair_imp<Q23std19allocator<7UV_RECT>,Ui,1> |
| 0x00484fa0 | 132 | WEAK | std::__vector_imp<15C_ParticleInUse,Q23std28allocator<15C_ParticleInUse>,0> | erase | erase__Q23std70__vector_imp<15C_ParticleInUse,Q23std28allocator<15C_ParticleInUse>,0>FP15C_ParticleInUse |
| 0x00485030 | 1180 | WEAK | std::__vector_imp<15C_ParticleInUse,Q23std28allocator<15C_ParticleInUse>,0> | insert | insert__Q23std70__vector_imp<15C_ParticleInUse,Q23std28allocator<15C_ParticleInUse>,0>FP15C_ParticleInUseUiRC15C_ParticleInUse |
| 0x004854d0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std28allocator<15C_ParticleInUse>,Ui,1> | swap | swap__Q310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15C_ParticleInUse>,Ui,1>FRQ310Metrowerks7details62compressed_pair_imp<Q23std28allocator<15C_ParticleInUse>,Ui,1> |
| 0x004854f0 | 72 | WEAK | std::__vector_deleter<7UV_RECT,Q23std19allocator<7UV_RECT>> | clear | clear__Q23std54__vector_deleter<7UV_RECT,Q23std19allocator<7UV_RECT>>Fv |
| 0x00485540 | 104 | WEAK | std::__vector_deleter<7UV_RECT,Q23std19allocator<7UV_RECT>> | {dtor} | __dt__Q23std54__vector_deleter<7UV_RECT,Q23std19allocator<7UV_RECT>>Fv |
| 0x004855b0 | 72 | WEAK | std::__vector_deleter<15C_ParticleInUse,Q23std28allocator<15C_ParticleInUse>> | clear | clear__Q23std72__vector_deleter<15C_ParticleInUse,Q23std28allocator<15C_ParticleInUse>>Fv |
| 0x00485600 | 104 | WEAK | std::__vector_deleter<15C_ParticleInUse,Q23std28allocator<15C_ParticleInUse>> | {dtor} | __dt__Q23std72__vector_deleter<15C_ParticleInUse,Q23std28allocator<15C_ParticleInUse>>Fv |
| 0x00485670 | 84 | GLOBAL | C_cache_base_block_ex | {ctor} | __ct__21C_cache_base_block_exFv |
| 0x004856d0 | 12 | 13 | Metrowerks::details::compressed_pair_imp<Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>,Ui,1> | {ctor} | __ct__Q310Metrowerks7details89compressed_pair_imp<Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>,Ui,1>FUi |
| 0x004856e0 | 112 | GLOBAL | C_cache_base_block_ex | {dtor} | __dt__21C_cache_base_block_exFv |
| 0x00485750 | 512 | GLOBAL | C_cache_base_block_ex | Close | Close__21C_cache_base_block_exFv |
| 0x00485950 | 3852 | GLOBAL | C_cache_base_block_ex | Open | Open__21C_cache_base_block_exFP7C_chunkP9I3D_scene |
| 0x00486860 | 8 | 13 | C_cache_base_block_ex::S_base_model | {ctor} | __ct__Q221C_cache_base_block_ex12S_base_modelFv |
| 0x00486870 | 8 | 13 | C_cache_base_block_ex::S_base_block | {ctor} | __ct__Q221C_cache_base_block_ex12S_base_blockFv |
| 0x00486880 | 168 | GLOBAL | C_cache_base_block_ex | GetNumModelVisual | GetNumModelVisual__21C_cache_base_block_exFP9I3D_modelPP10I3D_object |
| 0x00486930 | 276 | 13 | C_cache_base_block_ex::S_hlp$2451_m_cache_ex_cpp | cbVisCount | cbVisCount__Q221C_cache_base_block_ex25S_hlp$2451_m_cache_ex_cppFP9I3D_framePiPP10I3D_object |
| 0x00486a50 | 1844 | GLOBAL | C_cache_base_block_ex | OpenModel | OpenModel__21C_cache_base_block_exFPCc |
| 0x00487190 | 84 | 13 | std::__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0> | {dtor} | __dt__Q23std124__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0>Fv |
| 0x004871f0 | 20 | 13 | std::__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>> | {ctor} | __ct__Q23std126__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>>Fv |
| 0x00487210 | 472 | GLOBAL | C_cache_base_block_ex | Tick | Tick__21C_cache_base_block_exFUi |
| 0x004873f0 | 932 | 13 | C_cache_base_block_ex | ActivateBlock | ActivateBlock__21C_cache_base_block_exFUi |
| 0x004877a0 | 228 | 13 | C_cache_base_block_ex | DeactivateBlock | DeactivateBlock__21C_cache_base_block_exFUi |
| 0x00487890 | 36 | 13 | C_cache_base_block_ex | IsModelOnlyObject | IsModelOnlyObject__21C_cache_base_block_exFUs |
| 0x004878c0 | 884 | 13 | C_cache_base_block_ex | ShowObject | ShowObject__21C_cache_base_block_exFUs |
| 0x00487c40 | 712 | LOCAL |  | TransparentModel__FP9I3D_frameb | TransparentModel__FP9I3D_frameb |
| 0x00487f10 | 48 | 13 | C_cache_base_block_ex | HideObject | HideObject__21C_cache_base_block_exFP10I3D_object |
| 0x00487f40 | 1256 | 13 | C_cache_base_block_ex | ShowModel | ShowModel__21C_cache_base_block_exFUsPUs |
| 0x00488430 | 232 | 13 | C_cache_base_block_ex | HideModel | HideModel__21C_cache_base_block_exFUsUs |
| 0x00488520 | 276 | GLOBAL | C_cache_base_block_ex | ClearUnusedModels | ClearUnusedModels__21C_cache_base_block_exFv |
| 0x00488640 | 1052 | WEAK | std::__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0> | insert | insert__Q23std124__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0>FPQ221C_cache_base_block_ex17S_cache_model_oneUiRCQ221C_cache_base_block_ex17S_cache_model_one |
| 0x00488a60 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,Ui,1> | swap | swap__Q310Metrowerks7details89compressed_pair_imp<Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,Ui,1>FRQ310Metrowerks7details89compressed_pair_imp<Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,Ui,1> |
| 0x00488a80 | 1628 | WEAK | std::__vector_imp<Q221C_cache_base_block_ex17S_cache_model_set,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>,0> | insert | insert__Q23std124__vector_imp<Q221C_cache_base_block_ex17S_cache_model_set,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>,0>FPQ221C_cache_base_block_ex17S_cache_model_setUiRCQ221C_cache_base_block_ex17S_cache_model_set |
| 0x004890e0 | 20 | 13 | Metrowerks::details::compressed_pair_imp<Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>,Ui,1> | swap | swap__Q310Metrowerks7details89compressed_pair_imp<Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>,Ui,1>FRQ310Metrowerks7details89compressed_pair_imp<Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>,Ui,1> |
| 0x00489100 | 72 | 13 | std::__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0> | operator= | __as__Q23std124__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0>FRCQ23std124__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0> |
| 0x00489150 | 20 | 13 | std::__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0> | end | end__Q23std124__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0>CFv |
| 0x00489170 | 8 | 13 | std::__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0> | begin | begin__Q23std124__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0>CFv |
| 0x00489180 | 20 | 13 | std::__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>> | {ctor} | __ct__Q23std126__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>>FRCQ23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one> |
| 0x004891a0 | 8 | 13 | std::__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>> | alloc | alloc__Q23std126__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>>CFv |
| 0x004891b0 | 104 | WEAK | std::__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>> | {dtor} | __dt__Q23std126__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>>Fv |
| 0x00489220 | 132 | WEAK | std::__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_set,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>> | clear | clear__Q23std126__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_set,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>>Fv |
| 0x004892b0 | 104 | WEAK | std::__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_set,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>> | {dtor} | __dt__Q23std126__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_set,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_set>>Fv |
| 0x00489320 | 268 | 13 | std::__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0> | init<PCQ221C_cache_base_block_ex17S_cache_model_one> | init<PCQ221C_cache_base_block_ex17S_cache_model_one>__Q23std124__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0>FPCQ221C_cache_base_block_ex17S_cache_model_onePCQ221C_cache_base_block_ex17S_cache_model_oneQ23std20forward_iterator_tag |
| 0x00489430 | 568 | 13 | std::__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0> | do_assign<PCQ221C_cache_base_block_ex17S_cache_model_one> | do_assign<PCQ221C_cache_base_block_ex17S_cache_model_one>__Q23std124__vector_imp<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>,0>FPCQ221C_cache_base_block_ex17S_cache_model_onePCQ221C_cache_base_block_ex17S_cache_model_oneQ23std20forward_iterator_tag |
| 0x00489670 | 72 | WEAK | std::__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>> | clear | clear__Q23std126__vector_deleter<Q221C_cache_base_block_ex17S_cache_model_one,Q23std55allocator<Q221C_cache_base_block_ex17S_cache_model_one>>Fv |
| 0x004896c0 | 848 | GLOBAL | g_collision | LineInConvexDynamics | LineInConvexDynamics__11g_collisionFR11tCLineParamP24tDynamicCollObjectConvex |
| 0x00489a10 | 1008 | GLOBAL | g_collision | SphereInConvexDynamics | SphereInConvexDynamics__11g_collisionFR13tCSphereParamP24tDynamicCollObjectConvex |
| 0x00489e00 | 848 | GLOBAL | g_collision | LineInConvexStatic | LineInConvexStatic__11g_collisionFR11tCLineParamP15g_volume_convex |
| 0x0048a150 | 1128 | GLOBAL | g_collision | SphereInConvexStatic | SphereInConvexStatic__11g_collisionFR13tCSphereParamP15g_volume_convex |
| 0x0048a5c0 | 164 | GLOBAL | G_GarageClass | Clear | Clear__13G_GarageClassFv |
| 0x0048a670 | 144 | GLOBAL | G_GarageClass | AddCarByIdx | AddCarByIdx__13G_GarageClassFUiUi |
| 0x0048a700 | 300 | GLOBAL | G_GarageClass | AddCarByActor | AddCarByActor__13G_GarageClassFP5C_car |
| 0x0048a830 | 112 | GLOBAL | G_GarageClass | RemoveCar | RemoveCar__13G_GarageClassFPc |
| 0x0048a8a0 | 88 | GLOBAL | G_GarageClass | Vysum | Vysum__13G_GarageClassFP5C_car |
| 0x0048a900 | 88 | GLOBAL | G_GarageClass | AutoVybuchlo | AutoVybuchlo__13G_GarageClassFP5C_car |
| 0x0048a960 | 2672 | GLOBAL | G_GarageClass | GameInit | GameInit__13G_GarageClassFv |
| 0x0048b3d0 | 1332 | GLOBAL | G_GarageClass | GameDone | GameDone__13G_GarageClassFPQ23std42vector<P5C_car,Q23std18allocator<P5C_car>> |
| 0x0048b910 | 728 | GLOBAL | G_GarageClass | GarageShowResultsProc | GarageShowResultsProc__13G_GarageClassFv |
| 0x0048bbf0 | 728 | GLOBAL | G_GarageClass | CarManager | CarManager__13G_GarageClassFv |
| 0x0048bed0 | 8 | GLOBAL | G_GarageClass | GenerateCars | GenerateCars__13G_GarageClassFv |
| 0x0048bee0 | 8 | GLOBAL | G_GarageClass | ReleaseCars | ReleaseCars__13G_GarageClassFv |
| 0x0048bef0 | 12 | GLOBAL | G_GarageClass | ShowResults | ShowResults__13G_GarageClassFv |
| 0x0048bf00 | 20 | GLOBAL | G_GarageClass | CarIsStolen | CarIsStolen__13G_GarageClassFP5C_car |
| 0x0048bf20 | 8 | GLOBAL | G_GarageClass | CarCantGetOut | CarCantGetOut__13G_GarageClassFP5C_car |
| 0x0048bf30 | 12 | GLOBAL | G_GarageClass | AddLastStolen | AddLastStolen__13G_GarageClassFv |
| 0x0048bf40 | 60 | GLOBAL | G_GarageClass | IsCarIn | IsCarIn__13G_GarageClassFP5C_car |
| 0x0048bf80 | 108 | GLOBAL | G_GarageClass | EnableSteal | EnableSteal__13G_GarageClassFUc |
| 0x0048bff0 | 76 | GLOBAL | G_GarageClass | CanSteal | CanSteal__13G_GarageClassFPUc |
| 0x0048c040 | 136 | GLOBAL | G_GarageClass | CanStart | CanStart__13G_GarageClassFPUc |
| 0x0048c0d0 | 172 | GLOBAL | G_GarageClass | SaveGameLoad | SaveGameLoad__13G_GarageClassFP19G_SaveGame_GameHead |
| 0x0048c180 | 248 | GLOBAL | G_GarageClass | SaveGameSave | SaveGameSave__13G_GarageClassFP19G_SaveGame_GameHead |
| 0x0048c280 | 240 | GLOBAL | G_GarageClass | EnumCarsForChange | EnumCarsForChange__13G_GarageClassFPQ23std32vector<Ui,Q23std13allocator<Ui>>P5C_carUi |
| 0x0048c370 | 224 | GLOBAL | G_GarageClass | EnumCars | EnumCars__13G_GarageClassFPQ23std32vector<Ui,Q23std13allocator<Ui>>P5C_carUi |
| 0x0048c450 | 280 | GLOBAL | G_GarageClass | EnumCarsVect | EnumCarsVect__13G_GarageClassFPQ23std32vector<Ui,Q23std13allocator<Ui>>PQ23std42vector<P5C_car,Q23std18allocator<P5C_car>>Ui |
| 0x0048c570 | 80 | GLOBAL | G_GarageClass | GetNumOfCars | GetNumOfCars__13G_GarageClassFP5C_car |
| 0x0048c5c0 | 220 | GLOBAL | G_GarageClass | GetParkableCar | GetParkableCar__13G_GarageClassFv |
| 0x0048c6a0 | 252 | GLOBAL | G_GarageClass | GetChangeableCar | GetChangeableCar__13G_GarageClassFv |
| 0x0048c7a0 | 416 | GLOBAL | G_GarageClass | ParkCar | ParkCar__13G_GarageClassFP5C_car |
| 0x0048c940 | 1200 | GLOBAL | G_GarageClass | ChangeCar | ChangeCar__13G_GarageClassFP5C_carUi |
| 0x0048cdf0 | 760 | GLOBAL | G_GarageClass | TakeCar | TakeCar__13G_GarageClassFUi |
| 0x0048d0f0 | 844 | GLOBAL | G_GarageClass | DestroyCar | DestroyCar__13G_GarageClassFP5C_car |
| 0x0048d440 | 464 | GLOBAL | G_GarageClass | HlidejPocet | HlidejPocet__13G_GarageClassFP5C_carib |
| 0x0048d610 | 476 | GLOBAL | G_GarageClass | HlidejPocetVect | HlidejPocetVect__13G_GarageClassFPQ23std42vector<P5C_car,Q23std18allocator<P5C_car>>ib |
| 0x0048d7f0 | 384 | GLOBAL | G_GarageClass | HlidejPocetHard | HlidejPocetHard__13G_GarageClassFPQ23std42vector<P5C_car,Q23std18allocator<P5C_car>>i |
| 0x0048d970 | 3616 | GLOBAL | G_GarageClass | UdelejBum | UdelejBum__13G_GarageClassFv |
| 0x0048e790 | 1000 | GLOBAL | G_GarageClass | DoGarageMenu | DoGarageMenu__13G_GarageClassFbUi |
| 0x0048eb80 | 1060 | GLOBAL | G_GarageClass | DoFreerideMenu | DoFreerideMenu__13G_GarageClassFv |
| 0x0048efb0 | 68 | 13 | G_GarageClass | {dtor} | __dt__13G_GarageClassFv |
| 0x0048f000 | 56 | GLOBAL | GM_GarageComplete | OnKey | OnKey__17GM_GarageCompleteFUcUi |
| 0x0048f040 | 236 | GLOBAL | GM_GarageComplete | OnClick | OnClick__17GM_GarageCompleteFUi |
| 0x0048f130 | 228 | GLOBAL | GM_GarageGarage | OnCreate | OnCreate__15GM_GarageGarageFv |
| 0x0048f220 | 52 | GLOBAL | GM_GarageGarage | OnKey | OnKey__15GM_GarageGarageFUcUi |
| 0x0048f260 | 252 | GLOBAL | GM_GarageGarage | OnClick | OnClick__15GM_GarageGarageFUi |
| 0x0048f360 | 120 | GLOBAL | GM_GarageGarage | OnCloseChild | OnCloseChild__15GM_GarageGarageFP7GM_MenuUiUi |
| 0x0048f3e0 | 440 | GLOBAL | GM_GarageSelect | OnCreate | OnCreate__15GM_GarageSelectFv |
| 0x0048f5a0 | 116 | GLOBAL | GM_GarageSelect | OnClose | OnClose__15GM_GarageSelectFv |
| 0x0048f620 | 1192 | GLOBAL | GM_GarageSelect | OnTick | OnTick__15GM_GarageSelectFUi |
| 0x0048fad0 | 120 | GLOBAL | GM_GarageSelect | OnKey | OnKey__15GM_GarageSelectFUcUi |
| 0x0048fb50 | 84 | GLOBAL | GM_GarageSelect | OnClick | OnClick__15GM_GarageSelectFUi |
| 0x0048fbb0 | 320 | GLOBAL | GM_GarageFull | OnCreate | OnCreate__13GM_GarageFullFv |
| 0x0048fcf0 | 116 | GLOBAL | GM_GarageFull | OnClose | OnClose__13GM_GarageFullFv |
| 0x0048fd70 | 276 | GLOBAL | GM_GarageFull | OnTick | OnTick__13GM_GarageFullFUi |
| 0x0048fe90 | 132 | GLOBAL | GM_GarageFull | OnKey | OnKey__13GM_GarageFullFUcUi |
| 0x0048ff20 | 84 | GLOBAL | GM_GarageFull | OnClick | OnClick__13GM_GarageFullFUi |
| 0x0048ff80 | 540 | GLOBAL | GM_GarageSummary | OnCreate | OnCreate__16GM_GarageSummaryFv |
| 0x004901a0 | 56 | GLOBAL | GM_GarageSummary | OnKey | OnKey__16GM_GarageSummaryFUcUi |
| 0x004901e0 | 8 | GLOBAL | GM_GarageSummary | OnClick | OnClick__16GM_GarageSummaryFUi |
| 0x004901f0 | 96 | 13 | GM_GarageSummary | {dtor} | __dt__16GM_GarageSummaryFv |
| 0x00490250 | 44 | 13 | GM_GarageSummary | OnBeforeCreate | OnBeforeCreate__16GM_GarageSummaryFv |
| 0x00490280 | 124 | 13 | GM_GarageFull | {dtor} | __dt__13GM_GarageFullFv |
| 0x00490300 | 12 | 13 | GM_GarageFull | OnBeforeCreate | OnBeforeCreate__13GM_GarageFullFv |
| 0x00490310 | 124 | 13 | GM_GarageSelect | {dtor} | __dt__15GM_GarageSelectFv |
| 0x00490390 | 36 | 13 | GM_GarageSelect | OnBeforeCreate | OnBeforeCreate__15GM_GarageSelectFv |
| 0x004903c0 | 124 | 13 | GM_GarageGarage | {dtor} | __dt__15GM_GarageGarageFv |
| 0x00490440 | 12 | 13 | GM_GarageGarage | OnBeforeCreate | OnBeforeCreate__15GM_GarageGarageFv |
| 0x00490450 | 96 | 13 | GM_GarageComplete | {dtor} | __dt__17GM_GarageCompleteFv |
| 0x004904b0 | 12 | 13 | GM_GarageComplete | OnBeforeCreate | OnBeforeCreate__17GM_GarageCompleteFv |
| 0x00490690 | 576 | LOCAL |  | _fpadd_parts | _fpadd_parts |
| 0x00490da0 | 276 | GLOBAL |  | __fpcmp_parts_d | __fpcmp_parts_d |
| 0x004911c0 | 324 | GLOBAL |  | Ascii2Sjis__FUc | Ascii2Sjis__FUc |
| 0x00491310 | 164 | GLOBAL |  | AsciiString2Sjis__FPUcPUs | AsciiString2Sjis__FPUcPUs |
| 0x004913c0 | 184 | GLOBAL |  | _GetDirEntry__FPc | _GetDirEntry__FPc |
| 0x00491480 | 248 | GLOBAL |  | MC_UploadIcons__Fv | MC_UploadIcons__Fv |
| 0x00491580 | 548 | GLOBAL |  | MC_CheckMemCards__FRi | MC_CheckMemCards__FRi |
| 0x004917b0 | 36 | GLOBAL |  | _clear_scan_info__Fv | _clear_scan_info__Fv |
| 0x004917e0 | 2624 | GLOBAL |  | MC_ScanSaveHeads__FPcPviiPiPcPcii | MC_ScanSaveHeads__FPcPviiPiPcPcii |
| 0x00492220 | 16 | GLOBAL |  | _clear_save_data__Fv | _clear_save_data__Fv |
| 0x00492230 | 4532 | GLOBAL |  | MC_SaveData___FPcPviiPcPci | MC_SaveData___FPcPviiPcPci |
| 0x004933f0 | 256 | GLOBAL |  | MC_GetCardNumber__FPcPc | MC_GetCardNumber__FPcPc |
| 0x004934f0 | 12 | GLOBAL |  | _clear_load_data__Fv | _clear_load_data__Fv |
| 0x00493500 | 848 | GLOBAL |  | MC_LoadData___FPcPviPc | MC_LoadData___FPcPviPc |
| 0x00493850 | 20 | GLOBAL |  | _clear_delete_data__Fv | _clear_delete_data__Fv |
| 0x00493870 | 1348 | GLOBAL |  | MC_DeleteData___FPcPc | MC_DeleteData___FPcPc |
| 0x00493dc0 | 12 | GLOBAL |  | _clear_format_data__Fv | _clear_format_data__Fv |
| 0x00493dd0 | 348 | GLOBAL |  | MC_Format___Fi | MC_Format___Fi |
| 0x004bbe80 | 24 | LOCAL |  | __sinit_iostream.cpp | __sinit_iostream.cpp |
| 0x004bbea0 | 56 | LOCAL |  | __sinit_locale.cpp | __sinit_locale.cpp |
| 0x004bbee0 | 868 | LOCAL |  | __sinit___mem_dbg__.cpp | __sinit___mem_dbg__.cpp |
| 0x004bc250 | 912 | LOCAL |  | __sinit__m_cache.cpp | __sinit__m_cache.cpp |
| 0x004bc5e0 | 988 | LOCAL |  | __sinit_C_actor.cpp | __sinit_C_actor.cpp |
| 0x004bc9c0 | 868 | LOCAL |  | __sinit_C_actor_shot.cpp | __sinit_C_actor_shot.cpp |
| 0x004bcd30 | 868 | LOCAL |  | __sinit_C_airplane.cpp | __sinit_C_airplane.cpp |
| 0x004bd0a0 | 928 | LOCAL |  | __sinit_C_animations.cpp | __sinit_C_animations.cpp |
| 0x004bd440 | 868 | LOCAL |  | __sinit_C_blood.cpp | __sinit_C_blood.cpp |
| 0x004bd7b0 | 868 | LOCAL |  | __sinit_C_bottle.cpp | __sinit_C_bottle.cpp |
| 0x004bdb20 | 868 | LOCAL |  | __sinit_C_box_new.cpp | __sinit_C_box_new.cpp |
| 0x004bde90 | 908 | LOCAL |  | __sinit_C_bridge.cpp | __sinit_C_bridge.cpp |
| 0x004be220 | 960 | LOCAL |  | __sinit_C_car.cpp | __sinit_C_car.cpp |
| 0x004be5e0 | 868 | LOCAL |  | __sinit_C_car_ex.cpp | __sinit_C_car_ex.cpp |
| 0x004be950 | 868 | LOCAL |  | __sinit_C_cartridge.cpp | __sinit_C_cartridge.cpp |
| 0x004becc0 | 868 | LOCAL |  | __sinit_c_clock.cpp | __sinit_c_clock.cpp |
| 0x004bf030 | 868 | LOCAL |  | __sinit_C_crash_obj.cpp | __sinit_C_crash_obj.cpp |
| 0x004bf3a0 | 868 | LOCAL |  | __sinit_C_detector.cpp | __sinit_C_detector.cpp |
| 0x004bf710 | 868 | LOCAL |  | __sinit_C_dog.cpp | __sinit_C_dog.cpp |
| 0x004bfa80 | 868 | LOCAL |  | __sinit_C_door.cpp | __sinit_C_door.cpp |
| 0x004bfdf0 | 868 | LOCAL |  | __sinit_C_driver.cpp | __sinit_C_driver.cpp |
| 0x004c0160 | 868 | LOCAL |  | __sinit_C_drop_in_weapon.cpp | __sinit_C_drop_in_weapon.cpp |
| 0x004c04d0 | 868 | LOCAL |  | __sinit_C_DropOut.cpp | __sinit_C_DropOut.cpp |
| 0x004c0840 | 868 | LOCAL |  | __sinit_C_ff_manager.cpp | __sinit_C_ff_manager.cpp |
| 0x004c0bb0 | 868 | LOCAL |  | __sinit_C_grenade.cpp | __sinit_C_grenade.cpp |
| 0x004c0f20 | 868 | LOCAL |  | __sinit_C_human.cpp | __sinit_C_human.cpp |
| 0x004c1290 | 868 | LOCAL |  | __sinit_C_irenka.cpp | __sinit_C_irenka.cpp |
| 0x004c1600 | 868 | LOCAL |  | __sinit_C_movement.cpp | __sinit_C_movement.cpp |
| 0x004c1970 | 1040 | LOCAL |  | __sinit_C_person_traffic.cpp | __sinit_C_person_traffic.cpp |
| 0x004c1d80 | 868 | LOCAL |  | __sinit_C_PhThings.cpp | __sinit_C_PhThings.cpp |
| 0x004c20f0 | 868 | LOCAL |  | __sinit_C_player.cpp | __sinit_C_player.cpp |
| 0x004c2460 | 948 | LOCAL |  | __sinit_C_program.cpp | __sinit_C_program.cpp |
| 0x004c2820 | 868 | LOCAL |  | __sinit_C_public_physics.cpp | __sinit_C_public_physics.cpp |
| 0x004c2b90 | 868 | LOCAL |  | __sinit_C_pumper.cpp | __sinit_C_pumper.cpp |
| 0x004c2f00 | 868 | LOCAL |  | __sinit_c_racecam.cpp | __sinit_c_racecam.cpp |
| 0x004c3270 | 868 | LOCAL |  | __sinit_C_racing_optimal_path.cpp | __sinit_C_racing_optimal_path.cpp |
| 0x004c35e0 | 1116 | LOCAL |  | __sinit_C_rail_generator.cpp | __sinit_C_rail_generator.cpp |
| 0x004c3a40 | 868 | LOCAL |  | __sinit_C_rail_traffic.cpp | __sinit_C_rail_traffic.cpp |
| 0x004c3db0 | 1116 | LOCAL |  | __sinit_C_railway.cpp | __sinit_C_railway.cpp |
| 0x004c4210 | 908 | LOCAL |  | __sinit_C_rmorf.cpp | __sinit_C_rmorf.cpp |
| 0x004c45a0 | 868 | LOCAL |  | __sinit_C_shvestky.cpp | __sinit_C_shvestky.cpp |
| 0x004c4910 | 868 | LOCAL |  | __sinit_C_sledovacka.cpp | __sinit_C_sledovacka.cpp |
| 0x004c4c80 | 868 | LOCAL |  | __sinit_C_smoke_trail.cpp | __sinit_C_smoke_trail.cpp |
| 0x004c4ff0 | 1236 | LOCAL |  | __sinit_C_trf_c.cpp | __sinit_C_trf_c.cpp |
| 0x004c54d0 | 868 | LOCAL |  | __sinit_C_turnout.cpp | __sinit_C_turnout.cpp |
| 0x004c5840 | 1220 | LOCAL |  | __sinit_C_Vehicle.cpp | __sinit_C_Vehicle.cpp |
| 0x004c5d10 | 868 | LOCAL |  | __sinit_C_Vehicle_effects.cpp | __sinit_C_Vehicle_effects.cpp |
| 0x004c6080 | 868 | LOCAL |  | __sinit_C_Vehicle_kernel.cpp | __sinit_C_Vehicle_kernel.cpp |
| 0x004c63f0 | 868 | LOCAL |  | __sinit_C_Vehicle_sounds.cpp | __sinit_C_Vehicle_sounds.cpp |
| 0x004c6760 | 868 | LOCAL |  | __sinit_C_wagon.cpp | __sinit_C_wagon.cpp |
| 0x004c6ad0 | 996 | LOCAL |  | __sinit_console.cpp | __sinit_console.cpp |
| 0x004c6ec0 | 1012 | LOCAL |  | __sinit_g_c_glob.cpp | __sinit_g_c_glob.cpp |
| 0x004c72c0 | 868 | LOCAL |  | __sinit_g_c_line.cpp | __sinit_g_c_line.cpp |
| 0x004c7630 | 868 | LOCAL |  | __sinit_g_c_linebs.cpp | __sinit_g_c_linebs.cpp |
| 0x004c79a0 | 868 | LOCAL |  | __sinit_g_c_sph.cpp | __sinit_g_c_sph.cpp |
| 0x004c7d10 | 928 | LOCAL |  | __sinit_g_camera.cpp | __sinit_g_camera.cpp |
| 0x004c80b0 | 868 | LOCAL |  | __sinit_g_colls.cpp | __sinit_g_colls.cpp |
| 0x004c8420 | 936 | LOCAL |  | __sinit_g_dabing.cpp | __sinit_g_dabing.cpp |
| 0x004c87d0 | 996 | LOCAL |  | __sinit_g_indicators.cpp | __sinit_g_indicators.cpp |
| 0x004c8bc0 | 948 | LOCAL |  | __sinit_g_loadsave.cpp | __sinit_g_loadsave.cpp |
| 0x004c8f80 | 928 | LOCAL |  | __sinit_g_m_carc.cpp | __sinit_g_m_carc.cpp |
| 0x004c9320 | 1916 | LOCAL |  | __sinit_g_m_comps.cpp | __sinit_g_m_comps.cpp |
| 0x004c9aa0 | 1556 | LOCAL |  | __sinit_g_m_comps2.cpp | __sinit_g_m_comps2.cpp |
| 0x004ca0c0 | 1460 | LOCAL |  | __sinit_g_m_compsm.cpp | __sinit_g_m_compsm.cpp |
| 0x004ca680 | 868 | LOCAL |  | __sinit_g_m_func.cpp | __sinit_g_m_func.cpp |
| 0x004ca9f0 | 868 | LOCAL |  | __sinit_g_m_psx.cpp | __sinit_g_m_psx.cpp |
| 0x004cad60 | 868 | LOCAL |  | __sinit_g_m_race.cpp | __sinit_g_m_race.cpp |
| 0x004cb0d0 | 868 | LOCAL |  | __sinit_g_m_start.cpp | __sinit_g_m_start.cpp |
| 0x004cb440 | 868 | LOCAL |  | __sinit_g_material.cpp | __sinit_g_material.cpp |
| 0x004cb7b0 | 876 | LOCAL |  | __sinit_g_menu.cpp | __sinit_g_menu.cpp |
| 0x004cbb20 | 868 | LOCAL |  | __sinit_g_shot.cpp | __sinit_g_shot.cpp |
| 0x004cbe90 | 912 | LOCAL |  | __sinit_g_textdb.cpp | __sinit_g_textdb.cpp |
| 0x004cc220 | 984 | LOCAL |  | __sinit_game.cpp | __sinit_game.cpp |
| 0x004cc600 | 868 | LOCAL |  | __sinit_jp_ai_actor_history.cpp | __sinit_jp_ai_actor_history.cpp |
| 0x004cc970 | 892 | LOCAL |  | __sinit_jp_ai_entity.cpp | __sinit_jp_ai_entity.cpp |
| 0x004cccf0 | 868 | LOCAL |  | __sinit_jp_ai_groups.cpp | __sinit_jp_ai_groups.cpp |
| 0x004cd060 | 868 | LOCAL |  | __sinit_jp_ai_checkbuild.cpp | __sinit_jp_ai_checkbuild.cpp |
| 0x004cd3d0 | 892 | LOCAL |  | __sinit_jp_ai_checkpoints.cpp | __sinit_jp_ai_checkpoints.cpp |
| 0x004cd750 | 1172 | LOCAL |  | __sinit_jp_ai_main.cpp | __sinit_jp_ai_main.cpp |
| 0x004cdbf0 | 868 | LOCAL |  | __sinit_jp_ai_prislusnici.cpp | __sinit_jp_ai_prislusnici.cpp |
| 0x004cdf60 | 868 | LOCAL |  | __sinit_jp_ai_segments.cpp | __sinit_jp_ai_segments.cpp |
| 0x004ce2d0 | 868 | LOCAL |  | __sinit_Load_progress.cpp | __sinit_Load_progress.cpp |
| 0x004ce640 | 1848 | LOCAL |  | __sinit_Main.cpp | __sinit_Main.cpp |
| 0x004ced80 | 64 | LOCAL |  | __sinit_MemoryCard.cpp | __sinit_MemoryCard.cpp |
| 0x004cedc0 | 988 | LOCAL |  | __sinit_Mission.cpp | __sinit_Mission.cpp |
| 0x004cf1a0 | 868 | LOCAL |  | __sinit_multiloop.cpp | __sinit_multiloop.cpp |
| 0x004cf510 | 868 | LOCAL |  | __sinit_pathweb.cpp | __sinit_pathweb.cpp |
| 0x004cf880 | 868 | LOCAL |  | __sinit_racing.cpp | __sinit_racing.cpp |
| 0x004cfbf0 | 868 | LOCAL |  | __sinit_RamCompress.cpp | __sinit_RamCompress.cpp |
| 0x004cff60 | 868 | LOCAL |  | __sinit_road_web.cpp | __sinit_road_web.cpp |
| 0x004d02d0 | 972 | LOCAL |  | __sinit_tools.cpp | __sinit_tools.cpp |
| 0x004d06a0 | 868 | LOCAL |  | __sinit_tshift.cpp | __sinit_tshift.cpp |
| 0x004d0a10 | 868 | LOCAL |  | __sinit_VE_BreakageGlass.cpp | __sinit_VE_BreakageGlass.cpp |
| 0x004d0d80 | 868 | LOCAL |  | __sinit_VE_Debris.cpp | __sinit_VE_Debris.cpp |
| 0x004d10f0 | 868 | LOCAL |  | __sinit_Winstuff.cpp | __sinit_Winstuff.cpp |
| 0x004d1460 | 236 | LOCAL |  | __sinit_rw_data.cpp | __sinit_rw_data.cpp |
| 0x004d1550 | 64 | LOCAL |  | __sinit_SPR_VU1Draw.cpp | __sinit_SPR_VU1Draw.cpp |
| 0x004d1590 | 64 | LOCAL |  | __sinit_Draw.cpp | __sinit_Draw.cpp |
| 0x004d15d0 | 64 | LOCAL |  | __sinit_LSI.cpp | __sinit_LSI.cpp |
| 0x004d1610 | 64 | LOCAL |  | __sinit_LSI_file.cpp | __sinit_LSI_file.cpp |
| 0x004d1650 | 64 | LOCAL |  | __sinit_lsi_globals.cpp | __sinit_lsi_globals.cpp |
| 0x004d1690 | 64 | LOCAL |  | __sinit_LSI_system.cpp | __sinit_LSI_system.cpp |
| 0x004d16d0 | 64 | LOCAL |  | __sinit_LSIDevice.cpp | __sinit_LSIDevice.cpp |
| 0x004d1710 | 64 | LOCAL |  | __sinit_LSIIdxBuff.cpp | __sinit_LSIIdxBuff.cpp |
| 0x004d1750 | 64 | LOCAL |  | __sinit_LSITexture.cpp | __sinit_LSITexture.cpp |
| 0x004d1790 | 64 | LOCAL |  | __sinit_LSIVertBuff.cpp | __sinit_LSIVertBuff.cpp |
| 0x004d17d0 | 124 | LOCAL |  | __sinit_LSIVifBuff.cpp | __sinit_LSIVifBuff.cpp |
| 0x004d1850 | 64 | LOCAL |  | __sinit_VU1_GSRegs.cpp | __sinit_VU1_GSRegs.cpp |
| 0x004d1890 | 64 | LOCAL |  | __sinit_LSI_SPR.cpp | __sinit_LSI_SPR.cpp |
| 0x004d18d0 | 64 | LOCAL |  | __sinit_FlexMemMan.cpp | __sinit_FlexMemMan.cpp |
| 0x004d1910 | 64 | LOCAL |  | __sinit_LSI_PSS.cpp | __sinit_LSI_PSS.cpp |
| 0x004d1950 | 64 | LOCAL |  | __sinit_LSI_sound_David.cpp | __sinit_LSI_sound_David.cpp |
| 0x004d1990 | 64 | LOCAL |  | __sinit_TexMemMan.cpp | __sinit_TexMemMan.cpp |
| 0x004d19d0 | 64 | LOCAL |  | __sinit_I3d_animation.cpp | __sinit_I3d_animation.cpp |
| 0x004d1a10 | 64 | LOCAL |  | __sinit_I3d_area.cpp | __sinit_I3d_area.cpp |
| 0x004d1a50 | 64 | LOCAL |  | __sinit_I3d_billboard.cpp | __sinit_I3d_billboard.cpp |
| 0x004d1a90 | 64 | LOCAL |  | __sinit_I3d_camera.cpp | __sinit_I3d_camera.cpp |
| 0x004d1ad0 | 64 | LOCAL |  | __sinit_i3d_collision.cpp | __sinit_i3d_collision.cpp |
| 0x004d1b10 | 1504 | LOCAL |  | __sinit_I3d_driver.cpp | __sinit_I3d_driver.cpp |
| 0x004d20f0 | 64 | LOCAL |  | __sinit_I3d_dummy.cpp | __sinit_I3d_dummy.cpp |
| 0x004d2130 | 124 | LOCAL |  | __sinit_I3d_frame.cpp | __sinit_I3d_frame.cpp |
| 0x004d21b0 | 64 | LOCAL |  | __sinit_I3d_interpolator.cpp | __sinit_I3d_interpolator.cpp |
| 0x004d21f0 | 64 | LOCAL |  | __sinit_I3d_joint.cpp | __sinit_I3d_joint.cpp |
| 0x004d2230 | 64 | LOCAL |  | __sinit_I3d_light.cpp | __sinit_I3d_light.cpp |
| 0x004d2270 | 64 | LOCAL |  | __sinit_I3d_lit_object.cpp | __sinit_I3d_lit_object.cpp |
| 0x004d22b0 | 64 | LOCAL |  | __sinit_I3d_material.cpp | __sinit_I3d_material.cpp |
| 0x004d22f0 | 64 | LOCAL |  | __sinit_I3d_math.cpp | __sinit_I3d_math.cpp |
| 0x004d2330 | 64 | LOCAL |  | __sinit_I3d_mesh.cpp | __sinit_I3d_mesh.cpp |
| 0x004d2370 | 64 | LOCAL |  | __sinit_I3d_mirror.cpp | __sinit_I3d_mirror.cpp |
| 0x004d23b0 | 208 | LOCAL |  | __sinit_I3d_model.cpp | __sinit_I3d_model.cpp |
| 0x004d2480 | 64 | LOCAL |  | __sinit_I3d_morph.cpp | __sinit_I3d_morph.cpp |
| 0x004d24c0 | 64 | LOCAL |  | __sinit_I3d_object.cpp | __sinit_I3d_object.cpp |
| 0x004d2500 | 64 | LOCAL |  | __sinit_I3d_occluder.cpp | __sinit_I3d_occluder.cpp |
| 0x004d2540 | 144 | LOCAL |  | __sinit_I3d_particle.cpp | __sinit_I3d_particle.cpp |
| 0x004d25d0 | 176 | LOCAL |  | __sinit_I3d_portal.cpp | __sinit_I3d_portal.cpp |
| 0x004d2680 | 64 | LOCAL |  | __sinit_I3d_pose.cpp | __sinit_I3d_pose.cpp |
| 0x004d26c0 | 64 | LOCAL |  | __sinit_I3d_projector.cpp | __sinit_I3d_projector.cpp |
| 0x004d2700 | 208 | LOCAL |  | __sinit_I3d_scene.cpp | __sinit_I3d_scene.cpp |
| 0x004d27d0 | 64 | LOCAL |  | __sinit_I3d_sector.cpp | __sinit_I3d_sector.cpp |
| 0x004d2810 | 124 | LOCAL |  | __sinit_I3d_shadow.cpp | __sinit_I3d_shadow.cpp |
| 0x004d2890 | 64 | LOCAL |  | __sinit_I3d_singlemesh.cpp | __sinit_I3d_singlemesh.cpp |
| 0x004d28d0 | 64 | LOCAL |  | __sinit_I3d_singlemorph.cpp | __sinit_I3d_singlemorph.cpp |
| 0x004d2910 | 64 | LOCAL |  | __sinit_I3d_sound.cpp | __sinit_I3d_sound.cpp |
| 0x004d2950 | 124 | LOCAL |  | __sinit_I3d_target.cpp | __sinit_I3d_target.cpp |
| 0x004d29d0 | 64 | LOCAL |  | __sinit_I3d_user.cpp | __sinit_I3d_user.cpp |
| 0x004d2a10 | 64 | LOCAL |  | __sinit_I3d_visual.cpp | __sinit_I3d_visual.cpp |
| 0x004d2a50 | 64 | LOCAL |  | __sinit_I3d_volume.cpp | __sinit_I3d_volume.cpp |
| 0x004d2a90 | 124 | LOCAL |  | __sinit_lightmap.cpp | __sinit_lightmap.cpp |
| 0x004d2b10 | 64 | LOCAL |  | __sinit_Load.cpp | __sinit_Load.cpp |
| 0x004d2b50 | 124 | LOCAL |  | __sinit_load4ds.cpp | __sinit_load4ds.cpp |
| 0x004d2bd0 | 64 | LOCAL |  | __sinit_Load5ds.cpp | __sinit_Load5ds.cpp |
| 0x004d2c10 | 304 | LOCAL |  | __sinit_render.cpp | __sinit_render.cpp |
| 0x004d2d40 | 124 | LOCAL |  | __sinit_T&l.cpp | __sinit_T&l.cpp |
| 0x004d2dc0 | 320 | LOCAL |  | __sinit_Globals.cpp | __sinit_Globals.cpp |
| 0x004d2f00 | 540 | LOCAL |  | __sinit_IGraph.cpp | __sinit_IGraph.cpp |
| 0x004d3120 | 64 | LOCAL |  | __sinit_IShow.cpp | __sinit_IShow.cpp |
| 0x004d3160 | 64 | LOCAL |  | __sinit_ITexture.cpp | __sinit_ITexture.cpp |
| 0x004d31a0 | 64 | LOCAL |  | __sinit_IS_3Dlistener.cpp | __sinit_IS_3Dlistener.cpp |
| 0x004d31e0 | 64 | LOCAL |  | __sinit_IS_3Dsound.cpp | __sinit_IS_3Dsound.cpp |
| 0x004d3220 | 64 | LOCAL |  | __sinit_IS_driver.cpp | __sinit_IS_driver.cpp |
| 0x004d3260 | 208 | LOCAL |  | __sinit_IS_sound.cpp | __sinit_IS_sound.cpp |
| 0x004d3330 | 64 | LOCAL |  | __sinit_IS_source.cpp | __sinit_IS_source.cpp |
| 0x004d3370 | 116 | LOCAL |  | __sinit_VBManager_PS2.cpp | __sinit_VBManager_PS2.cpp |
| 0x004d33f0 | 112 | LOCAL |  | __sinit_IBManager_PS2.cpp | __sinit_IBManager_PS2.cpp |
| 0x004d3460 | 868 | LOCAL |  | __sinit_g_vid.cpp | __sinit_g_vid.cpp |
| 0x004d37d0 | 868 | LOCAL |  | __sinit_g_inventory.cpp | __sinit_g_inventory.cpp |
| 0x004d3b40 | 868 | LOCAL |  | __sinit_C_Particle.cpp | __sinit_C_Particle.cpp |
| 0x004d3eb0 | 1100 | LOCAL |  | __sinit_C_ParticleEmitter.cpp | __sinit_C_ParticleEmitter.cpp |
| 0x004d4300 | 912 | LOCAL |  | __sinit_C_ParticleManager.cpp | __sinit_C_ParticleManager.cpp |
| 0x004d4690 | 912 | LOCAL |  | __sinit__m_cache_ex.cpp | __sinit__m_cache_ex.cpp |
| 0x004d4a20 | 868 | LOCAL |  | __sinit_g_c_convex.cpp | __sinit_g_c_convex.cpp |
| 0x004d4d90 | 916 | LOCAL |  | __sinit_g_garage.cpp | __sinit_g_garage.cpp |
| 0x004d5130 | 868 | LOCAL |  | __sinit_g_m_garage.cpp | __sinit_g_m_garage.cpp |
| 0x004d54a0 | 64 | LOCAL |  | __sinit_LSI_MCard.cpp | __sinit_LSI_MCard.cpp |
