# vim: sw=4
project('pcm', ['c', 'cpp'],
  version : '202104',
  default_options : ['warning_level=3', 'cpp_std=c++14'])

conf_data = configuration_data()
cc = meson.get_compiler('c')
cxx = meson.get_compiler('cpp')
is_linux = host_machine.system() == 'linux'
null_dep = dependency('', required: false)

libpcm_srcs = [
    'msr.cpp',
    'cpucounters.cpp',
    'pci.cpp',
    'mmio.cpp',
    'bw.cpp',
    'utils.cpp',
    'topology.cpp',
    'dashboard.cpp',
    'debug.cpp',
    'threadpool.cpp',
    'resctrl.cpp',
]

targets = [
    'pcm',
    'pcm-numa',
    'pcm-latency',
    'pcm-power',
    'pcm-sensor',
    'pcm-msr',
    'pcm-memory',
    'pcm-tsx',
    'pcm-pcie',
    'pcm-core',
    'pcm-iio',
    'pcm-lspci',
    'pcm-pcicfg',
    'pcm-mmio',
]

thread_dep = dependency('threads')
perf_dep = null_dep
dl_dep = null_dep
openssl_dep = null_dep

if not cc.has_function('dlopen')
    dl_dep = cc.find_library('dl', required: host_machine.system() != 'windows')
endif

# Linux
if is_linux
    if cxx.has_header('linux/perf_event.h', required: get_option('perf'))
        conf_data.set('PCM_USE_PERF', true)
    endif

    _openssl_dep = dependency('openssl', required: get_option('openssl'))
    if _openssl_dep.found()
        openssl_dep = _openssl_dep
        conf_data.set('USE_SSL', true)
    endif

    if get_option('dev-mem-access') == true
        conf_data.set('PCM_USE_PCI_MM_LINUX', true)
    endif
endif

foreach flag: conf_data.keys()
    add_project_arguments('-D' + flag, language: 'cpp')
endforeach

deps = [thread_dep, perf_dep, openssl_dep]

pcm_static = static_library('pcm',
    sources: libpcm_srcs,
    dependencies: deps
)

pcm_shared = shared_library('pcm',
    sources: ['pcm-core.cpp'],
    link_with: pcm_static,
    cpp_args: ['-DPCM_SILENT'],
    dependencies: deps,
    install: true
)

foreach target: targets
    executable(target + '.x',
        sources: [target + '.cpp'],
        link_with: pcm_static,
        install_dir: get_option('sbindir'),
        install: true
    )
endforeach

if is_linux
    executable('pcm-sensor-server.x',
        sources: ['pcm-sensor-server.cpp', 'favicon.ico.h'],
        link_with: pcm_static,
        install_dir: get_option('sbindir'),
        install: true
    )
endif

if get_option('memoptest')
    executable('memoptest.x',
        sources: ['memoptest.cpp']
    )
endif

if get_option('dashboardtest')
    executable('dashboardtest.x',
        sources: ['dashboardtest.cpp'],
        link_with: pcm_static
    )
endif

if get_option('examples')
    executable('c_example.x',
        sources: ['c_example.c'],
        build_rpath: '$ORIGIN/',
        c_args: ['-DPCM_DYNAMIC_LIB'],
        dependencies: [dl_dep],
    )
    executable('c_example_shlib.x',
        sources: ['c_example.c'],
        dependencies: [thread_dep],
        link_with: pcm_shared
    )
endif

subdir('daemon/client')
subdir('daemon/daemon')
