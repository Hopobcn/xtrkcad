FILE(GLOB HEADERS *.h)

SET(SOURCES
	gtkbitmap.c
	gtkbutton.c
	gtkcolor.c
	gtkfilsel.c
	gtkfont.c
	gtkhelp.c
	gtklist.c
	gtkmenu.c
	gtkmisc.c
	gtksimple.c
	gtksingle.c
	gtksplash.c
	gtktext.c
	gtkwindow.c
	psprint.c
	wpref.c
	)

IF(XTRKCAD_USE_GTK_CAIRO)
	SET(SOURCES ${SOURCES} gtkdraw-cairo.c)
ELSE(XTRKCAD_USE_GTK_CAIRO)
	SET(SOURCES ${SOURCES} gtkdraw.c)
ENDIF(XTRKCAD_USE_GTK_CAIRO)

SET_SOURCE_FILES_PROPERTIES(wpref.c PROPERTIES COMPILE_FLAGS -DEXPORT=)

INCLUDE_DIRECTORIES(${XTrkCAD_BINARY_DIR})
INCLUDE_DIRECTORIES(${GTK_INCLUDE_DIRS})
INCLUDE_DIRECTORIES(${GTK_WEBKIT_INCLUDE_DIRS})

ADD_LIBRARY(xtrkcad-wlib ${HEADERS} ${SOURCES})
TARGET_LINK_LIBRARIES(xtrkcad-wlib ${GTK_LIBRARIES})
TARGET_LINK_LIBRARIES(xtrkcad-wlib ${GTK_WEBKIT_LIBRARIES})
