#!python
import os
import toml

config = toml.load('config.toml')

proj_name = list(config)[0]
bin_dir = config[proj_name]["bin_dir"]
source_dir = config[proj_name]["source_dir"]
base_dir = config[proj_name]["project_dir"]
cpp_bindings = config[proj_name]["cpp_bindings"]

GODOT_CPP_ENTRY = """{}

using namespace godot;

extern "C" void GDN_EXPORT godot_gdnative_init(godot_gdnative_init_options *o) {{
  Godot::gdnative_init(o);
}}

extern "C" void GDN_EXPORT
godot_gdnative_terminate(godot_gdnative_terminate_options *o) {{
  Godot::gdnative_terminate(o);
}}

extern "C" void GDN_EXPORT godot_nativescript_init(void *handle) {{
  Godot::nativescript_init(handle);
  {}
}}
"""

GDNLIB_STRING = f"""
[general]

singleton=false
load_once=true
symbol_prefix="godot_"
reloadable=false

[entry]

X11.64="res://{bin_dir}/x11/lib{proj_name}.so"
Windows.64="res://{bin_dir}/win64/lib{proj_name}.dll"
OSX.64="res://{bin_dir}/osx/lib{proj_name}.dylib"

[dependencies]

X11.64=[]
Windows.64=[]
OSX.64=[]"""

GDNS_STRING = """[gd_resource type="NativeScript" load_steps=2 format=2]

[ext_resource path="res://{}/{}.gdnlib" type="GDNativeLibrary" id=1]

[resource]
resource_name = "{}"
class_name = "{}"
library = ExtResource( 1 )"""

def gen_bindings(sources):
    with open(os.path.join(base_dir, bin_dir, f"{proj_name}.gdnlib"), "w") as f:
        f.write(GDNLIB_STRING)

    for binding in config[proj_name]["bindings"]:
        file = os.path.join(base_dir, bin_dir, binding.lower() + ".gdns")
        with open(file, "w") as f:
            f.write(GDNS_STRING.format(bin_dir, proj_name, binding.lower(), binding))
    
    headers = Glob(source_dir + "/*.hpp")
    include_str = ""
    for header in headers:
        include_str += f"#include \"{os.path.basename(header.name)}\"\n"

    register_str = ""
    for class_ in config[proj_name]["bindings"]:
        register_str += f"register_class<{class_}>();\n"
    
    with open(os.path.join(source_dir, "entry.cpp"), "w") as f:
        f.write(GODOT_CPP_ENTRY.format(include_str, register_str))




opts = Variables([], ARGUMENTS)

# Gets the standard flags CC, CCX, etc.
env = DefaultEnvironment()

# Define our options
opts.Add(EnumVariable('target', "Compilation target", 'debug', ['d', 'debug', 'r', 'release']))
opts.Add(EnumVariable('platform', "Compilation platform", '', ['', 'windows', 'x11', 'linux', 'osx']))
opts.Add(EnumVariable('p', "Compilation target, alias for 'platform'", '', ['', 'windows', 'x11', 'linux', 'osx']))
opts.Add(BoolVariable('use_llvm', "Use the LLVM / Clang compiler", 'no'))
opts.Add(PathVariable('target_path', 'The path where the lib is installed.', os.path.join(base_dir, bin_dir)))
opts.Add(PathVariable('target_name', 'The library name.', 'lib' + proj_name, PathVariable.PathAccept))


godot_headers_path = cpp_bindings + "/godot-headers/"
cpp_library = "libgodot-cpp"

bits = 64

opts.Update(env)

std = config[proj_name]["compiler"]["standard"]

if env['use_llvm']:
    env['CC'] = 'clang'
    env['CXX'] = 'clang++'

if env['p'] != '':
    env['platform'] = env['p']

if env['platform'] == '':
    print("No valid target platform selected.")
    quit()

env['target_path'] += '/'

if env['platform'] == "osx":
    env['target_path'] += 'osx/'
    cpp_library += '.osx'
    env.Append(CCFLAGS=['-arch', 'x86_64'])
    env.Append(CXXFLAGS=[f'-std={std}'])
    env.Append(LINKFLAGS=['-arch', 'x86_64'])
    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS=['-g', '-O2'])
    else:
        env.Append(CCFLAGS=['-g', '-O3'])

elif env['platform'] in ('x11', 'linux'):
    env['target_path'] += 'x11/'
    cpp_library += '.linux'
    env.Append(CCFLAGS=['-fPIC'])
    env.Append(CXXFLAGS=[f'-std={std}'])
    if env['target'] in ('debug', 'd'):
        env.Append(CCFLAGS=['-g3', '-Og'])
    else:
        env.Append(CCFLAGS=['-g', '-O3'])

elif env['platform'] == "windows":
    env['target_path'] += 'win64/'
    cpp_library += '.windows'
    env.Append(ENV=os.environ)

    env.Append(CPPDEFINES=['WIN32', '_WIN32', '_WINDOWS', '_CRT_SECURE_NO_WARNINGS'])
    env.Append(CCFLAGS=['-W3', '-GR'])
    env.Append(CXXFLAGS=f'/std:{std}')
    if env['target'] in ('debug', 'd'):
        env.Append(CPPDEFINES=['_DEBUG'])
        env.Append(CCFLAGS=['-EHsc', '-MDd', '-ZI'])
        env.Append(LINKFLAGS=['-DEBUG'])
    else:
        env.Append(CPPDEFINES=['NDEBUG'])
        env.Append(CCFLAGS=['-O2', '-EHsc', '-MD'])

if env['target'] in ('debug', 'd'):
    cpp_library += '.debug'
else:
    cpp_library += '.release'

cpp_library += '.' + str(bits)

env.Append(CPPPATH=['.', godot_headers_path, config[proj_name]["compiler"]["include_paths"]])
env.Append(LIBPATH=[os.path.join(cpp_bindings, 'bin')])
env.Append(LIBS=[cpp_library])

env.Append(CPPPATH=[config[proj_name]["source_dir"]])

gen_bindings(Glob(config[proj_name]["sources"]));

library = env.SharedLibrary(target=env['target_path'] + env['target_name'] , source=Glob(config[proj_name]["sources"]))

Default(library)

Help(opts.GenerateHelpText(env))
