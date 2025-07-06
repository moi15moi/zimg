project('zimg', 'cpp',
  license: 'WTFPL',
  meson_version: '>=1.0.0',
  default_options: ['cpp_std=c++17'],
  version: '3.0.5',
)

# Detect platform and CPU architecture
cpu_family = host_machine.cpu_family()

x86 = cpu_family.startswith('x86')
arm = cpu_family.startswith('arm') or cpu_family.startswith('aarch64')

# Compiler flags
cc = meson.get_compiler('cpp')
add_project_arguments('-fvisibility=hidden', language: 'cpp')

# Options
testapp = get_option('testapp')
example = get_option('example')
unit_test = get_option('unit-test')
debug = get_option('debug')
simd = get_option('simd')

# Debug flags
if debug
  add_project_arguments('-O0', '-g3', '-ggdb', language: 'cpp')
else
  add_project_arguments('-DNDEBUG', language: 'cpp')
endif

# SIMD
neon_cflags = []
hsw_cflags = []
skx_cflags = []
clx_cflags = []

enable_arm_simd = false
enable_x86_simd = false

if simd
  if arm
    enable_arm_simd = true
    add_project_arguments('-DZIMG_ARM', language: 'cpp')
  elif x86
    enable_x86_simd = true
    add_project_arguments('-DZIMG_X86', language: 'cpp')

    if cc.has_argument('-mtune=haswell')
      hsw_cflags += ['-mtune=haswell']
    endif
    if cc.has_argument('-mtune=skylake-avx512')
      skx_cflags += ['-mtune=skylake-avx512']
    endif
    if cc.has_argument('-mtune=cascadelake')
      clx_cflags += ['-mtune=cascadelake']
    endif
  endif
endif

# Pthreads
threads_dep = dependency('threads', required: false)
if not threads_dep.found()
  warning('Unable to find pthread. Testapp will be single-threaded.')
endif

# Main library
zimg_sources = files(
  'src/zimg.cpp',
  'src/zimg_api.cpp',
  # Add other source files here
)

zimg_lib = library('zimg',
  zimg_sources,
  include_directories: include_directories('src'),
  dependencies: [threads_dep],
  install: true
)

# Optional programs
if testapp
  executable('zimgtest', 'tools/testapp.cpp',
    dependencies: [zimg_lib, threads_dep],
    install: true)
endif

if example
  executable('zimg_example', 'tools/example.cpp',
    dependencies: [zimg_lib],
    install: true)
endif

# Unit tests
if unit_test
  # Placeholder: You should define gtest manually or via a subproject
  gtest_dep = dependency('gtest', required: false)
  if gtest_dep.found()
    test_exe = executable('zimg_unittest',
      'test/test_zimg.cpp',
      dependencies: [zimg_lib, gtest_dep],
      install: false)
    test('unit', test_exe)
  else
    warning('GoogleTest not found; skipping unit tests')
  endif
endif

# pkg-config file
pkg = import('pkgconfig')
pkg.generate(
  name: 'zimg',
  description: 'A scaling, colorspace conversion, and pixel format conversion library',
  version: meson.project_version(),
  libraries: zimg_lib,
  filebase: 'zimg',
  subdirs: 'zimg'
)