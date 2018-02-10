add_library(
	swell
	STATIC
	commctrl.h
	shlobj.h
	swell-appstub-generic.cpp
	swell-appstub.mm
    # swell-dlg-generic.cpp
	swell-dlg.mm
	swell-dlggen.h
	swell-functions.h
    # swell-gdi-generic.cpp
	swell-gdi-internalpool.h
    # swell-gdi-lice.cpp
	swell-gdi.mm
	swell-ini.cpp
	swell-internal.h
    # swell-kb-generic.cpp
	swell-kb.mm
    # swell-menu-generic.cpp
	swell-menu.mm
	swell-menugen.h
    # swell-misc-generic.cpp
	swell-misc.mm
    # swell-miscdlg-generic.cpp
    # swell-miscdlg-gtk.cpp
	swell-miscdlg.mm
    # swell-modstub-generic.cpp
	swell-modstub.mm
	swell-types.h
    # swell-wnd-generic.cpp
	swell-wnd.mm
	swell.cpp
	swell.h
	swellappmain.h
	swellappmain.mm
	# test-gtk.cpp
	# test.cpp
	windows.h
)

