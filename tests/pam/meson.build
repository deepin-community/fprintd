subdir('services')

tests = [
    'test_pam_fprintd',
]

preloaded_libs = []
pam_tests_ld_preload = []

if address_sanitizer
    # ASAN has to be the first in list
    preloaded_libs += 'asan'
endif

preloaded_libs += 'pam_wrapper'

foreach libname: preloaded_libs
    lib = run_command(meson.get_compiler('c'),
        '-print-file-name=lib@0@.so'.format(libname)
    ).stdout().strip()

    # Support linker script files
    if run_command('grep', '-qI', '^INPUT', files(lib)).returncode() == 0
        out = run_command('cat', lib).stdout()
        lib = out.split('(')[1].split(')')[0].strip()
    endif

    if lib != '' and lib[0] == '/'
        message('Found library @0@ as @1@'.format(libname, lib))
        pam_tests_ld_preload += '@0@'.format(files(lib)[0])
    else
        tests = []
        warning('No library found for ' + libname + ', skipping PAM tests')
    endif
endforeach

foreach t: tests
    python_tests += [
        {
            'name': t,
            'file': files(meson.current_source_dir() / t + '.py')[0],
            'is_parallel': false,
            'env': [
                'TOPBUILDDIR=' + meson.build_root(),
                'TOPSRCDIR=' + meson.source_root(),
                'LD_PRELOAD=' + ' '.join(pam_tests_ld_preload),
                'PAM_WRAPPER=1',
                'PAM_WRAPPER_DEBUGLEVEL=2',
                'PAM_WRAPPER_SERVICE_DIR=' + meson.current_build_dir() / 'services',
                'G_DEBUG=fatal-warnings',
            ],
            'depends': [
                pam_fprintd,
                pam_service_file,
            ],
            'suite': ['PAM'],
        }
    ]
endforeach
