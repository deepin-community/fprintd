bash = find_program('bash')
dbus_interfaces = ['Manager', 'Device']
dbus_interfaces_files = []

foreach interface_name: dbus_interfaces
    interface = interface_name.to_lower()
    interface_file = interface + '.xml'
    dbus_interfaces_files += custom_target('dbus_interface_' + interface,
        input: interface_file,
        output: 'net.reactivated.Fprint.@0@.xml'.format(interface_name),
        command: ['cp', '@INPUT@', '@OUTPUT@'],
        install: true,
        install_dir: dbus_interfaces_dir,
    )
endforeach

# NOTE: We should pass "--glib-min-required 2.64" but cannot
fprintd_dbus_sources_base = gnome.gdbus_codegen('fprintd-dbus',
    sources: dbus_interfaces_files,
    autocleanup: 'all',
    interface_prefix: 'net.reactivated.Fprint.',
    namespace: 'FprintDBus',
    object_manager: true,
)

# FIXME: remove this and just use fprintd_dbus_sources when we're on glib 2.64
fprintd_dbus_sources = [
    fprintd_dbus_sources_base[1] # header file
]

fprintd_dbus_sources += custom_target('fprintd-dbus-interactive',
    input: fprintd_dbus_sources_base[0], # c file,
    output: 'fprintd-dbus-interactive.c',
    command: [
        find_program('patch'),
        '-p1',
        '--merge',
        '@INPUT@',
        files('dbus-interactive-auth.patch'),
        '-o', '@OUTPUT@',
    ])

fprintd_enum_files = gnome.mkenums_simple('fprintd-enums',
  sources: 'fprintd.h',
)

fprintd_deps = declare_dependency(
    include_directories: [
        include_directories('..'),
    ],
    sources: [
        fprintd_enum_files,
        fprintd_dbus_sources,
    ],
    dependencies: [
        glib_dep,
        gio_dep,
        gio_unix_dep,
        gmodule_dep,
        libfprint_dep,
        polkit_gobject_dep,
    ],
    compile_args: [
        '-DG_LOG_DOMAIN="@0@"'.format(meson.project_name()),
        '-DLOCALEDIR="@0@"'.format(localedir),
        '-DPLUGINDIR="@0@"'.format(fprintd_plugindir),
    ],
)

libfprintd_private = static_library('fprintd-private',
    sources: [
        'device.c',
        'fprintd.h',
        'manager.c',
    ],
    dependencies: fprintd_deps,
    gnu_symbol_visibility: 'hidden',
)

fprintd = executable('fprintd',
    sources: [
        'file_storage.c',
        'file_storage.h',
        'fprintd.h',
        'main.c',
        'storage.h',
    ],
    dependencies: fprintd_deps,
    link_with: libfprintd_private,
    gnu_symbol_visibility: 'hidden',
    install: true,
    install_dir: fprintd_installdir,
)
