docbook_xml_header = custom_target('docbook_xml_header',
    output: 'docbook-xml-header.xml',
    command: [
        'echo', '-n',
        '<?xml version="1.0"?>\n',
        '<!DOCTYPE refentry PUBLIC "-//OASIS//DTD DocBook XML V4.1.2//EN" "http://www.oasis-open.org/docbook/xml/4.1.2/docbookx.dtd">\n',
    ],
    capture: true,
)

dbus_interfaces_refs = []
foreach interface_file: dbus_interfaces_files
    basename = run_command('basename', interface_file.full_path(), '.xml').stdout().strip()
    dbus_interfaces_refs += custom_target(basename + '_ref',
        input: docbook_xml_header,
        output: basename + '.ref.xml',
        build_by_default: true,
        depends: interface_file,
        capture: true,
        command: [
            bash, '-c',
            'cat @INPUT@;' +
            xsltproc.path() + ' @0@/@1@ '.format(
                meson.source_root(),
                files('spec-to-docbook.xsl')[0]) +
            interface_file.full_path() + '| tail -n +2;',
        ],
    )
endforeach
