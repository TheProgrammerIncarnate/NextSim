if (FIPS_EMSCRIPTEN)
    add_definitions(-DSOKOL_GLES2)
    set(slang "glsl100")
elseif (FIPS_WINDOWS)
    add_definitions(-DSOKOL_D3D11)
    set(slang "hlsl5")
elseif (FIPS_OSX)
    add_definitions(-DSOKOL_METAL)
    if (FIPS_IOS)
        set(slang "metal_ios:metal_sim")
    else()
        set(slang "metal_macos")
    endif()
else()
    add_definitions(-DSOKOL_GLCORE33)
    set(slang "glsl330")
endif()

fips_begin_lib(sokol)
    fips_vs_warning_level(3)
    if (FIPS_OSX)
        fips_files(sokol.m)
        if (FIPS_IOS)
            fips_frameworks_osx(UIKit Metal MetalKit AudioToolbox)
        else()
            fips_frameworks_osx(Cocoa QuartzCore Metal MetalKit AudioToolbox)
        endif()
    else()
        fips_files(sokol.c)
        if (FIPS_ANDROID)
            fips_libs(GLESv3 EGL OpenSLES log android)
        elseif (FIPS_LINUX)
            fips_libs(X11 GL m dl asound)
        endif()
    endif()
fips_end_lib()
#Change this based on targetted chip
set(chippath "AY38500")
#Options:
#6502 -The legendary 6502
#AY38500 -Currently just a 6502 with a renamed netlist

fips_include_directories(unique/${chippath})
fips_begin_app(NextSim windowed)
    fips_vs_warning_level(3)
    fips_files(main.h main.c chipvis.c pick.c gfx.c sim.c trace.c util.c asm.c netlist_sim.c)
    fips_files(ui.cc ui_asm.cc)
    sokol_shader(chipvis.glsl ${slang})
	fips_dir(unique/${chippath})
	fips_files(support.c segdefs.c netlist.c nodenames.c)#No potential for unique source files besides these?
    fips_dir(res)
    fips_generate(FROM fonts.yml TYPE dump HEADER fonts.h)
    fips_generate(FROM markdown.yml TYPE dump HEADER markdown.h)
    fips_files(help_assembler.md about.md favicon.png)
    fips_deps(sokol imgui texteditor asmx)
fips_end_app()
