daemon_srcs = [
    'daemon.cpp',
    'main.cpp',
]

executable('pcm-daemon',
    sources: daemon_srcs,
    dependencies: [thread_dep],
    link_with: pcm_static,
    install: is_linux,
    install_dir: get_option('sbindir')
)
