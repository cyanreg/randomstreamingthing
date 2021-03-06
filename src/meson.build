# Base files
sources = [
    'txproto_main.c',

    # Sources
    'iosys_common.c',

    # Outputs
    'interface.c',
    'interface_common.c',

    # Muxing
    'muxing.c',

    # Filtering
    'filtering.c',

    # Encoding
    'encoding.c',

    # Misc
    'utils.c',
    'logging.c',
    'fifo_frame.c',
    'fifo_packet.c',

    builtin_lua_scripts,

    # Version
    vcs_tag(command: ['git', 'rev-parse', '--short', 'HEAD'],
                      input: 'version.c.in', output: 'version.c',
                      fallback: 'release')
]

# libendit
libedit = dependency('libedit', version: '>= 3.1', required: get_option('libedit'))
if libedit.found()
    dependencies += libedit
    sources += 'repl.c'
    conf.set('HAVE_LIBEDIT', 1)
endif

# libavdevice
libavdevice = dependency('libavdevice', version: '>= 58.9.100', required: get_option('libavdevice'))
if libavdevice.found()
    dependencies += libavdevice
    sources += 'iosys_lavd.c'
    conf.set('HAVE_LAVD', 1)
endif

# Pulseaudio
libpulse = dependency('libpulse', version: '>= 0.29.0', required: get_option('pulse'))
if libpulse.found()
    dependencies += libpulse
    sources += 'iosys_pulse.c'
    conf.set('HAVE_PULSEAUDIO', 1)
endif

# Wayland
wayland_scanner = find_program('wayland-scanner', required: get_option('wayland'))
wayland_deps = [ # Reorder with care, wayland_protocols expects them like this
    dependency('wayland-client', required: get_option('wayland')),
    dependency('wayland-cursor', required: get_option('wayland')),
    dependency('wayland-protocols', version: '>=1.14', required: get_option('wayland')),
    dependency('xkbcommon', version: '>= 0.3.0', required: get_option('wayland')),
    dependency('libdrm', version: '>=2.4.95', required: get_option('wayland')),
]

have_wayland = wayland_deps.length() == 5 and wayland_scanner.found() and has_memfd
if have_wayland
    subdir('wayland_protocols')
    dependencies += wayland_client_protos
    dependencies += wayland_deps.get(3)
    dependencies += wayland_deps.get(4)
    sources += 'iosys_wayland.c'
    sources += 'interface_wayland.c'
    sources += 'wayland_common.c'
    conf.set('HAVE_WAYLAND', 1)

    gbm_dep = dependency('gbm', version: '>=17.1.0', required: false)
    if gbm_dep.found()
        dependencies += gbm_dep
        conf.set('HAVE_GBM', 1)
    endif
endif

executable('txproto',
    install: true,
    sources: sources,
    dependencies: dependencies,
)

summary({
    'libedit': libedit.found(),
}, section: 'Interface', bool_yn: true)

summary({
    'libavdevice': libavdevice.found(),
    'pulse': libpulse.found(),
    'wayland': have_wayland,
}, section: 'I/O systems', bool_yn: true)
