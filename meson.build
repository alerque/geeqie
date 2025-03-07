# This file is a part of Geeqie project (https://www.geeqie.org/).
# Copyright (C) 2008 - 2022 The Geeqie Team
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.

# Meson default directories used in this project:
# prefix - /usr/local
# bindir - bin
# datadir - share

# Meson core options:
# buildtype
# debug

# Project expanded default directories:
# prefix        /usr/local/
# bindir        /usr/local/bin                          geeqie executable
# gq_bindir     /usr/local/lib/geeqie               *   plugins scripts
# datadir       /usr/local/share/
#               /usr/local/share/applications           geeqie.desktop
# [gq_]appdir   /usr/local/share/geeqie/            *   template.desktop
# desktopdir    /usr/local/share/geeqie/applications    plugin desktop files
# appdatadir    /usr/local/share/metainfo               org.geeqie.Geeqie.appdata.xml
# icondir       /usr/local/share/pixmaps                geeqie.png icon
# [gq_]helpdir  /usr/local/share/doc/geeqie         *   readme files etc.
# [gq_]htmldir  /usr/local/share/doc/geeqie/html    *   help files
# gq_localedir  /usr/locale/share/locale
# mandir1       /usr/local/share/man/man1               man page
# podir         project_root/po

# * See meson_options.txt file

project(
    'geeqie',
    'c',
    'cpp',
    version : run_command('./version.sh', check : true).stdout().strip(),
    license : ['GPL-2.0-or-later'],
    meson_version : '>=0.56.2',
    default_options : ['cpp_std=c++14', 'warning_level=3', 'buildtype=debugoptimized', 'cpp_link_args=-rdynamic']
)

# To inhibit warnings from the generated files icons_inline.h and ui_icons.h
add_global_arguments('-Wno-overlength-strings', language : 'c')

# To compile originally-C files as C++
add_global_arguments('-Wno-error=deprecated-declarations', language : 'cpp')
add_global_arguments('-Wno-error=sign-compare', language : 'cpp')
add_global_arguments('-Wno-error=return-type', language : 'cpp')
add_global_arguments('-Wno-error=literal-suffix', language : 'cpp')
add_global_arguments('-Wno-error=write-strings', language : 'cpp')

# Project requirements
project_sources = []
gnome = import('gnome')
thread_dep = dependency('threads')
cc = meson.get_compiler('c')
i18n = import('i18n')
fs = import('fs')
configuration_inc = include_directories('.')

# Extended stack trace using backward-app
option = get_option('devel')
if option.enabled()
    if cc.has_link_argument('-ldwarf')
        add_project_link_arguments('-ldwarf', language: 'cpp')
    endif
endif

# External programs
gdk_pixbuf_csource = find_program('gdk-pixbuf-csource', required : true)
glib_compile_resources = find_program('glib-compile-resources', required : true)
glib_genmarshal = find_program('glib-genmarshal', required : true)

option = get_option('git')
if not option.disabled()
    running_from_git = find_program('git', required: false).found() and fs.is_dir('.git')
else
    running_from_git = false
    summary({'git' : ['disabled - ChangeLog, ChangeLog.html, lua-api help file created:', false]}, section : 'Documentation', bool_yn : true)
endif

debug = get_option('debug')

# Note that main.cc sets prefix to the directory above where the executable is run from.
# This is to allow AppImages to be used

# These gq_* variables are paths relative to /prefix/,
# and are also used in defines in the source as GQ_*
if get_option('gq_appdir') == ''
    gq_appdir = join_paths(get_option('datadir'), 'geeqie')
else
    gq_appdir = get_option('gq_appdir')
endif

# This is not the same as Meson bindir
if get_option('gq_bindir') == ''
    gq_bindir = 'lib/geeqie'
else
    gq_bindir = get_option('gq_bindir')
endif

if get_option('gq_helpdir') == ''
    gq_helpdir = join_paths(get_option('datadir'), 'doc/geeqie')
else
    gq_helpdir = get_option('gq_helpdir')
endif

if get_option('gq_htmldir') == ''
    gq_htmldir = join_paths(get_option('datadir'), 'doc/geeqie/html')
else
    gq_htmldir = get_option('gq_htmldir')
endif

if get_option('gq_localedir') == ''
    gq_localedir = join_paths(get_option('datadir'), 'locale')
else
    gq_localedir = get_option('gq_localedir')
endif


# Set up the absolute directory paths used
prefix = get_option('prefix')
datadir = join_paths(prefix, get_option('datadir'))

# Installation paths are absolute
appdir = join_paths(prefix, gq_appdir)
appdatadir = join_paths(datadir, 'metainfo')
desktopdir = join_paths(datadir, meson.project_name(), 'applications')
helpdir = join_paths(prefix, gq_helpdir)
htmldir = join_paths(prefix, gq_htmldir)
icondir = join_paths(datadir, 'pixmaps')
mandir1 = join_paths(datadir, 'man', 'man1')

podir = join_paths(meson.project_source_root(), 'po')
scriptsdir = join_paths(meson.project_source_root(), 'scripts')

summary({'gq_appdir': gq_appdir,
        'gq_bindir': gq_helpdir,
        'gq_helpdir': gq_helpdir,
        'gq_htmldir': gq_htmldir,
        'gq_localedir': gq_localedir,
        }, section: 'Directories')

# Create the define constants used in the sources. Set via config.h.in
conf_data = configuration_data()
conf_data.set_quoted('VERSION', meson.project_version())
conf_data.set('DEBUG', debug)

option = get_option('gtk4')
if option.enabled()
    gtk_dep = dependency('gtk4', required: true)
    conf_data.set('HAVE_GTK4', 1)
else
    gtk_dep = dependency('gtk+-3.0', version : '>=3.24', required: true)
endif
glib_dep = dependency('glib-2.0', version : '>=2.52', required: true)

# Required only when backward-cpp is used
libdw_dep = []
libunwind_dep = []
option = get_option('devel')
if option.enabled()
    libdw_dep = dependency('libdw', required : true)
    if libdw_dep.found()
        libunwind_dep = dependency('libunwind', required : true)
        if libunwind_dep.found()
            conf_data.set('HAVE_DEVELOPER', 1)
            summary({'developer mode' : ['extended stacktrace:', true]}, section : 'Debugging', bool_yn : true)
        else
            summary({'developer mode' : ['libunwind not found. extended stacktrace:', false]}, section : 'Debugging', bool_yn : true)
        endif
    else
        summary({'developer mode' : ['libdw not found. extended stacktrace:', false]}, section : 'Debugging', bool_yn : true)
    endif
else
    summary({'developer mode' : ['extended stacktrace:', false]}, section : 'Debugging', bool_yn : true)
endif

# Required only for seg. fault stacktrace and backtrace debugging
option = get_option('execinfo')
libexecinfo_dep = []
if not option.disabled()
    result = cc.check_header('execinfo.h')
    if result
        # Include unconditionally dependency for NetBSD.
        libexecinfo_dep = cc.find_library('execinfo', required : false)
        conf_data.set('HAVE_EXECINFO_H', 1)
        summary({'execinfo' : ['stacktrace supported:', true]}, section : 'Debugging', bool_yn : true)
    else
        summary({'execinfo' : ['stacktrace supported:', false]}, section : 'Debugging', bool_yn : true)
    endif
else
    summary({'execinfo' : ['stacktrace supported:', false]}, section : 'Debugging', bool_yn : true)
endif

libarchive_dep = []
req_version = '>=3.4.0'
option = get_option('archive')
if not option.disabled()
    libarchive_dep = dependency('libarchive', version : req_version, required : get_option('archive'))
    if libarchive_dep.found()
        conf_data.set('HAVE_ARCHIVE', 1)
        summary({'archive' : ['archive files e.g. .zip supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'archive' : ['libarchive ' + req_version + ' not found - archive files e.g. .zip supported::', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'archive' : ['disabled - archive files e.g. .zip supported:', false]}, section : 'Configuration', bool_yn : true)
endif

lcms_dep = []
req_version = '>=2.0'
option = get_option('cms')
if not option.disabled()
    xxd = find_program('xxd', 'xxdi.pl', required : false)
    if xxd.found()
        lcms_dep = dependency('lcms2', version : req_version, required : get_option('cms'))
        if lcms_dep.found()
            conf_data.set('HAVE_LCMS', 1)
            conf_data.set('HAVE_LCMS2', 1)
            summary({'cms' : ['color management supported:', true]}, section : 'Configuration', bool_yn : true)
        else
            summary({'cms' : ['lcms2' + req_version + ' not found - color management supported:', false]}, section : 'Configuration', bool_yn : true)
        endif
    else
        summary({'cms' : ['xxd or xxdi.pl not found - color management supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'cms' : ['disabled - color management supported:', false]}, section : 'Configuration', bool_yn : true)
endif

ddjvuapi_dep = []
req_version = '>=2.5.27'
option = get_option('djvu')
if not option.disabled()
    ddjvuapi_dep = dependency('ddjvuapi', version : req_version, required : get_option('djvu'))
    if ddjvuapi_dep.found()
        conf_data.set('HAVE_DJVU', 1)
        summary({'djvu' : ['djvu files supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'djvu' : ['ddjvuapi ' + req_version + ' not found - djvu files supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'djvu' : ['disabled - djvu files supported:', false]}, section : 'Configuration', bool_yn : true)
endif

option = get_option('evince')
if not option.disabled()
    evince = find_program('evince', required : false)
    if evince.found()
        summary({'print preview' : ['print preview supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'print preview' : ['evince not found - print preview supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'print preview' : ['disabled - print preview supported:', false]}, section : 'Configuration', bool_yn : true)
endif

# Required only for seg. fault stacktrace and backtrace debugging
option = get_option('execinfo')
if not option.disabled()
    result = cc.check_header('execinfo.h')
    if result
        conf_data.set('HAVE_EXECINFO_H', 1)
        summary({'execinfo' : ['stacktrace supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'execinfo' : ['stacktrace supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'execinfo' : ['stacktrace supported:', false]}, section : 'Configuration', bool_yn : true)
endif

exiv2_dep = []
req_version = '>=0.11'
option = get_option('exiv2')
if not option.disabled()
    exiv2_dep = dependency('exiv2', version : req_version, required : get_option('exiv2'))
    if exiv2_dep.found()
        conf_data.set('HAVE_EXIV2', 1)
        summary({'exiv2' : ['image metadata processed by exiv2:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'exiv2' : ['exiv2 ' + req_version + ' not found - image data not processed by exiv2:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'exiv2' : ['disabled - image data processed by exiv2:', false]}, section : 'Configuration', bool_yn : true)
endif

champlain_dep = []
champlain_gtk_dep = []
clutter_dep = []
clutter_gtk_dep = []
req_version_champlain = '>=0.12'
req_version_champlain_gtk = '>=0.12'
req_version_clutter = '>=1.0'
req_version_clutter_gtk = '>=1.0'
option = get_option('gps-map')
if not option.disabled()
    champlain_dep = dependency('champlain-0.12', version : req_version_champlain, required : get_option('gps-map'))
    champlain_gtk_dep = dependency('champlain-gtk-0.12', version : req_version_champlain_gtk, required : get_option('gps-map'))
    if champlain_dep.found() and champlain_gtk_dep.found()
        clutter_dep = dependency('clutter-1.0', version : req_version_clutter, required : get_option('gps-map'))
        clutter_gtk_dep = dependency('clutter-gtk-1.0', version : req_version_clutter_gtk, required : get_option('gps-map'))
        if clutter_dep.found() and clutter_gtk_dep.found()
            conf_data.set('HAVE_CLUTTER', 1)
            conf_data.set('HAVE_LIBCHAMPLAIN', 1)
            conf_data.set('HAVE_LIBCHAMPLAIN_GTK', 1)
            summary({'gps-map' : ['GPS map displayed', true]}, section : 'Configuration', bool_yn : true)
        else
            if not clutter_dep.found()
                summary({'gps-map-clutter' : ['clutter-1.0 ' + req_version_clutter + ' not found - GPS map displayed:', false]}, section : 'Configuration', bool_yn : true)
            endif
            if not clutter_gtk_dep.found()
                summary({'gps-map-clutter-gtk' : ['clutter-gtk-1.0 ' + req_version_clutter_gtk + ' not found - GPS map displayed:', false]}, section : 'Configuration', bool_yn : true)
            endif
        endif
    else
        if not champlain_dep.found()
            summary({'gps-map-champlain' : ['champlain-0.12 ' + req_version_champlain + ' not found - GPS map displayed:', false]}, section : 'Configuration', bool_yn : true)
        endif
        if not champlain_gtk_dep.found()
            summary({'gps-map-champlain-gtk' : ['champlain-gtk-0.12 ' + req_version_champlain_gtk + ' not found - GPS map displayed:', false]}, section : 'Configuration', bool_yn : true)
        endif
    endif
else
    summary({'gps-map' : ['disabled - GPS map displayed:', false]}, section : 'Configuration', bool_yn : true)
endif

libheif_dep = []
req_version = '>=1.3.2'
option = get_option('heif')
if not option.disabled()
    libheif_dep = dependency('libheif', version : req_version, required : get_option('heif'))
    if libheif_dep.found()
        conf_data.set('HAVE_HEIF', 1)
        summary({'heif' : ['heif files supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'heif' : ['libheif ' + req_version + ' not found - heif files supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'heif' : ['disabled - heif files supported:', false]}, section : 'Configuration', bool_yn : true)
endif

libopenjp2_dep = []
req_version = '>=2.3.0'
option = get_option('j2k')
if not option.disabled()
    libopenjp2_dep = dependency('libopenjp2', version : req_version, required : get_option('j2k'))
    if libopenjp2_dep.found()
        conf_data.set('HAVE_J2K', 1)
        summary({'j2k' : ['j2k files supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'j2k' : ['libopenjp2 ' + req_version + ' not found - j2k files supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'j2k' : ['disabled - j2k files supported:', false]}, section : 'Configuration', bool_yn : true)
endif

libjpeg_dep = []
option = get_option('jpeg')
if not option.disabled()
libjpeg_dep = dependency('libjpeg', required : get_option('jpeg'))
    if libjpeg_dep.found()
        if cc.has_function('jpeg_destroy_decompress', dependencies : libjpeg_dep)
            conf_data.set('HAVE_JPEG', 1)
            summary({'jpeg' : ['jpeg files supported:', true]}, section : 'Configuration', bool_yn : true)
        else
            summary({'jpeg' : ['jpeg_destroy_decompress not found - jpeg files supported:', false]}, section : 'Configuration', bool_yn : true)
        endif
    else
        summary({'jpeg' : ['libjpeg: not found', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'jpeg' : ['disabled - jpeg files supported:', false]}, section : 'Configuration', bool_yn : true)
endif

libjxl_dep = []
req_version = '>=0.3.7'
option = get_option('jpegxl')
if not option.disabled()
    libjxl_dep = dependency('libjxl', version : req_version, required : get_option('jpegxl'))
    if libjxl_dep.found()
        conf_data.set('HAVE_JPEGXL', 1)
        summary({'jpegxl' : ['jpegxl files supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'jpegxl' : ['libjxl ' + req_version + ' not found - jpegxl files supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'jpegxl' : ['disabled - jpegxl files supported:', false]}, section : 'Configuration', bool_yn : true)
endif

libraw_dep = []
req_version = '>=0.20'
option = get_option('libraw')
if not option.disabled()
    libraw_dep = dependency('libraw', version : req_version, required : get_option('libraw'))
    if libraw_dep.found()
        conf_data.set('HAVE_RAW', 1)
        summary({'libraw' : ['.cr3 files supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'libraw' : ['libraw ' + req_version + ' not found - .cr3 files supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'libraw' : ['disabled - .cr3 files supported:', false]}, section : 'Configuration', bool_yn : true)
endif

lua_dep = []
req_version = '>=5.3'
option = get_option('lua')
if not option.disabled()
    foreach name : ['lua', 'lua5.3', 'lua-5.3', 'lua53']
        lua_dep = dependency(name, version: req_version, required: get_option('lua'))
        if lua_dep.found()
            break
        endif
    endforeach
    if lua_dep.found()
        conf_data.set('HAVE_LUA', 1)
        summary({'lua' : ['lua supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'lua' : ['lua ' + req_version + ' not found - lua supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'lua' : ['disabled - lua supported:', false]}, section : 'Configuration', bool_yn : true)
endif

# Install standard documents
option = get_option('pandoc')
if not option.disabled()
    pandoc = find_program('pandoc', required : false)
    if pandoc.found()
        readme_html = custom_target(
            'README.html',
            input: 'README.md',
            output: 'README.html',
            command: [pandoc, '@INPUT@', '-o', '@OUTPUT@'],
            install: true,
            install_dir: helpdir)

        summary({'README' : ['README.html created:', true]}, section : 'Documentation', bool_yn : true)
    else
        summary({'README' : ['pandoc not found - README.html created:', false]}, section : 'Documentation', bool_yn : true)
    endif
else
    summary({'pandoc' : ['disabled - README.html created:', false]}, section : 'Documentation', bool_yn : true)
endif
install_data('README.md', 'COPYING', 'TODO', install_dir : helpdir)

poppler_glib_dep = []
req_version = '>=0.62'
option = get_option('pdf')
if not option.disabled()
    poppler_glib_dep = dependency('poppler-glib', version : req_version, required : get_option('pdf'))
    if poppler_glib_dep.found()
        conf_data.set('HAVE_PDF', 1)
        summary({'pdf'  : ['pdf files supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'pdf' : ['poppler-glib ' + req_version + ' not found - pdf files supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'pdf' : ['disabled - pdf files supported:', false]}, section : 'Configuration', bool_yn : true)
endif

gspell_dep = []
req_version = '>=1.6'
option = get_option('spell')
if not option.disabled()
    gspell_dep = dependency('gspell-1', version : req_version, required: get_option('spell'))
    if gspell_dep.found()
        conf_data.set('HAVE_SPELL', 1)
        summary({'spell' : ['spelling checks enabled', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'spell' : ['gspell-1 ' + req_version + ' not found - spelling checks enabled', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'spell' : ['disabled - spelling checks enabled', false]}, section : 'Configuration', bool_yn : true)
endif

tiff_dep = []
option = get_option('tiff')
if not option.disabled()
    tiff_dep = dependency('libtiff-4', required: get_option('tiff'))
    if tiff_dep.found()
        if cc.has_function('TIFFClientOpen', dependencies : tiff_dep)
            conf_data.set('HAVE_TIFF', 1)
            summary({'tiff' : ['tiff files supported:', true]}, section : 'Configuration', bool_yn : true)
        else
            summary({'tiff' : ['TIFFClientOpen not found - tiff files supported:', false]}, section : 'Configuration', bool_yn : true)
        endif
    else
        summary({'tiff' : ['libtiff not found - tiff files supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'tiff' : ['disabled - tiff files supported:', false]}, section : 'Configuration', bool_yn : true)
endif

libffmpegthumbnailer_dep = []
req_version = '>=2.1.0'
option = get_option('videothumbnailer')
if not option.disabled()
    libffmpegthumbnailer_dep = dependency('libffmpegthumbnailer',
        version : req_version,
        required : get_option('videothumbnailer'))

    if libffmpegthumbnailer_dep.found()
        conf_data.set('HAVE_FFMPEGTHUMBNAILER', 1)
        summary({'videothumbnailer' : ['thumbnails of video files supported:', true]}, section : 'Configuration', bool_yn : true)

        result = cc.has_member('struct video_thumbnailer_struct', 'prefer_embedded_metadata', prefix : '#include <libffmpegthumbnailer/videothumbnailerc.h>')
        if result
            conf_data.set('HAVE_FFMPEGTHUMBNAILER_METADATA', 1)
        endif
        summary({'fmpegthumbnailer_metadata' : ['fmpegthumbnailer_metadata found:', result]}, section : 'Thumbnailer', bool_yn : true)

        result = cc.has_member('struct image_data_struct', 'image_data_width', prefix : '#include <libffmpegthumbnailer/videothumbnailerc.h>' )
        if result
            conf_data.set('HAVE_FFMPEGTHUMBNAILER_RGB', 1)
        endif
        summary({'fmpegthumbnailer_rgb' : ['fmpegthumbnailer_rgb found:', result]}, section : 'Thumbnailer', bool_yn : true)

        result = cc.has_function('video_thumbnailer_set_size', dependencies : libffmpegthumbnailer_dep)
        if result
            conf_data.set('HAVE_FFMPEGTHUMBNAILER_WH', 1)
        endif
        summary({'fmpegthumbnailer_set_size' : ['fmpegthumbnailer_set_size found:', result]}, section : 'Thumbnailer', bool_yn : true)
    else
        summary({'videothumbnailer' : ['libvideothumbnailer ' + req_version + ' not found - thumbnails of video files supported', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'videothumbnailer' : ['disabled -thumbnails of video files supported', false]}, section : 'Configuration', bool_yn : true)
endif

# libpixbufloader-webp is not loaded as part of libgdk-pixbuf. Just issue
# a warning if not installed
libwebp_dir = dependency('gdk-pixbuf-2.0', method: 'pkg-config').get_variable(pkgconfig: 'gdk_pixbuf_moduledir', internal: 'gdk_pixbuf_moduledir')

if libwebp_dir.contains('loaders')
    libwebp_dep = cc.find_library('pixbufloader-webp', dirs : libwebp_dir, required : false)
    if libwebp_dep.found()
        summary({'webp' : ['webp files supported:', true]}, section : 'Configuration', bool_yn : true)
    else
        summary({'webp' : ['webp-pixbuf-loader not installed - webp files supported:', false]}, section : 'Configuration', bool_yn : true)
    endif
else
    summary({'webp' : ['webp-pixbuf-loader not installed - webp files supported:', false]}, section : 'Configuration', bool_yn : true)
endif

# Check for nl_langinfo and _NL_TIME_FIRST_WEEKDAY
code = '''#include <langinfo.h>
#include<stdio.h>
int main (int argc, char ** argv) {
    char *c;
    c =  nl_langinfo(_NL_TIME_FIRST_WEEKDAY);
    return 0;
}'''
if cc.links(code, name : 'nl_langinfo and _NL_TIME_FIRST_WEEKDAY')
    conf_data.set('HAVE__NL_TIME_FIRST_WEEKDAY', 1)
    summary({'nl_langinfo' : ['first weekday depends on locale:', true]}, section : 'Documentation', bool_yn : true)
else
    summary({'nl_langinfo' : ['nl_langinfo not found - first weekday depends on locale:', false, 'first weekday defaults to Monday']}, section : 'Documentation', bool_yn : true)
endif

conf_data.set_quoted('GETTEXT_PACKAGE', meson.project_name())
conf_data.set_quoted('GQ_APPDIR', gq_appdir)
conf_data.set_quoted('GQ_BINDIR', gq_bindir)
conf_data.set_quoted('GQ_HELPDIR', gq_helpdir)
conf_data.set_quoted('GQ_HTMLDIR', gq_htmldir)
conf_data.set_quoted('GQ_LOCALEDIR', gq_localedir)

conf_data.set_quoted('PACKAGE', meson.project_name())
conf_data.set_quoted('PACKAGE_NAME', meson.project_name())
conf_data.set_quoted('PACKAGE_STRING', meson.project_version())
conf_data.set_quoted('PACKAGE_TARNAME', meson.project_name())
conf_data.set_quoted('PACKAGE_VERSION', meson.project_version())
conf_data.set_quoted('VERSION', meson.project_version())

configure_file(input : 'config.h.in',
               output : 'config.h',
               encoding : 'UTF-8',
               configuration : conf_data)

# For shellcheck on scripts
script_sources = []
subdir('scripts')

# For gtk builder checks on .ui files
ui_sources = []

# Process subdirs before the sources
subdir('po')
subdir('plugins')

# Generate the executable
subdir('src')

# Generate the help files
subdir('doc')

# Install other project files
if running_from_git
    cmd = [find_program('gen_changelog.sh'), meson.current_source_dir(), meson.current_build_dir()]
    custom_target(
        'ChangeLog',
        input: 'ChangeLog.gqview',
        output: ['ChangeLog', 'ChangeLog.html'],
        command: cmd,
        install: true,
        install_dir: helpdir)
    meson.add_dist_script(cmd)
    summary({'ChangeLog' : ['ChangeLog, ChangeLog.html created:', true]}, section : 'Documentation', bool_yn : true)
elif fs.exists('ChangeLog.html')
    install_data('ChangeLog', 'ChangeLog.html', install_dir: helpdir)
    summary({'ChangeLog' : ['ChangeLog, ChangeLog.html installed from dist:', true]}, section : 'Documentation', bool_yn : true)
endif

install_data('geeqie.png', install_dir : icondir)
install_data('geeqie.1', install_dir : mandir1)

i18n.merge_file(
    input : 'geeqie.desktop.in',
    output : 'geeqie.desktop',
    type : 'desktop',
    po_dir : podir,
    install : true,
    install_dir : join_paths(datadir, 'applications'))

i18n.merge_file(
    input : 'org.geeqie.Geeqie.appdata.xml.in',
    output : 'org.geeqie.Geeqie.appdata.xml',
    type : 'xml',
    po_dir : podir,
    install : true,
    install_dir : appdatadir)

configure_file(input: 'geeqie.spec.in', output: 'geeqie.spec', configuration: conf_data)

# Basic test of the executable
xvfb = find_program('xvfb-run', required : false)
if xvfb.found()
    test('Basic test', xvfb, args: ['--auto-servernum', geeqie_exe, '--version'], timeout: 100)
    summary({'xvfb' : ['Test runs:', true]}, section : 'Testing', bool_yn : true)
else
    summary({'xvfb' : ['Test runs:', false]}, section : 'Testing', bool_yn : true)
endif

# Shellcheck
shellcheck_exe = find_program('shellcheck', required : false)
script_sources += files('gen_changelog.sh',
'geeqie-install-debian.sh',
'version.sh')

if shellcheck_exe.found()
    foreach script : script_sources
        test('Shellcheck', shellcheck_exe, args: ['--norc', '--shell=sh', '--enable=add-default-case,avoid-nullary-conditions,check-unassigned-uppercase,deprecate-which,quote-safe-variables', script], timeout: 100)
    endforeach
    summary({'shellcheck' : ['Test runs:', true]}, section : 'Testing', bool_yn : true)
else
    summary({'shellcheck' : ['Test runs:', false]}, section : 'Testing', bool_yn : true)
endif

# GtkBuilder .ui check
if xvfb.found()
    gtk_builder_tool = find_program('gtk-builder-tool', required : false)
    if gtk_builder_tool.found()
        foreach ui_file : ui_sources
            test('UI Build', xvfb, args: ['--auto-servernum', gtk_builder_tool.full_path(), 'validate', ui_file], timeout: 100)
        endforeach
        summary({'gtk-builder-tool' : ['Test runs:', true]}, section : 'Testing', bool_yn : true)
    else
        summary({'gtk-builder-tool' : ['Test runs:', false]}, section : 'Testing', bool_yn : true)
    endif
else
    summary({'gtk-builder-tool xvfb' : ['Test runs:', false]}, section : 'Testing', bool_yn : true)
endif
