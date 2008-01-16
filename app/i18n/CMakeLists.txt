PROJECT(i18n)

# List of the supported locales.
SET(XTRKCAD_LOCALE_IDS
	fi
	)

# Misc variables
SET(XTC_LOCALE_TARGETS )
SET(POTFILE "${i18n_BINARY_DIR}/${GETTEXT_PACKAGE}.pot")
SET(XGTKEYWORDS
	--keyword=_
	--keyword=N_)

# Programs
FIND_PROGRAM(XTC_XGETTEXT xgettext)
FIND_PROGRAM(XTC_MSGINIT msginit)
FIND_PROGRAM(XTC_MSGMERGE msgmerge)
FIND_PROGRAM(XTC_MSGFMT msgfmt)
SET(XTC_TRANSLATOR "" CACHE FILEPATH "Translator program for editing the .po files.")
MARK_AS_ADVANCED(XTC_XGETTEXT XTC_MSGINIT XTC_MSGMERGE
                 XTC_MSGFMT XTC_TRANSLATOR)

# Create the list of the original source files.
FILE(GLOB_RECURSE XGT_SOURCE_FILES
	RELATIVE ${i18n_SOURCE_DIR}
	${XTrkCAD_SOURCE_DIR}/*.c
	${XTrkCAD_SOURCE_DIR}/*.h)

# Dynamically generated files, such as messages.h and bllnhlp.c, must be added
# to the list manually to make sure that everything works properly.
# "FILE(RELATIVE_PATH ..." is used to get rid of the absolute paths like
# "/home/my_funny_username/very_long_path_to_my_projects/source_file.c"
# in the .pot and .po files.
FILE(RELATIVE_PATH TEMP_FILENAME ${i18n_SOURCE_DIR} "${help_BINARY_DIR}/messages.h")
LIST(APPEND XGT_SOURCE_FILES ${TEMP_FILENAME})
FILE(RELATIVE_PATH TEMP_FILENAME ${i18n_SOURCE_DIR} "${XTrkCAD_BINARY_DIR}/app/bin/bllnhlp.c")
LIST(APPEND XGT_SOURCE_FILES ${TEMP_FILENAME})

# Extract strings and create xtrkcad.pot
ADD_CUSTOM_COMMAND(
	OUTPUT ${POTFILE}
	COMMAND ${XTC_XGETTEXT} -d ${GETTEXT_PACKAGE} ${XGTKEYWORDS} -s -o ${POTFILE} --from-code=ISO-8859-15 ${XGT_SOURCE_FILES}
	DEPENDS xtrkcad
	WORKING_DIRECTORY ${i18n_SOURCE_DIR}
	)

# Loop through the list of the supported locales
FOREACH(localeid ${XTRKCAD_LOCALE_IDS})
	SET(POFILE ${i18n_SOURCE_DIR}/${localeid}.po)
	SET(MOFILE_DIR ${i18n_BINARY_DIR}/${localeid}/LC_MESSAGES)
	SET(MOFILE ${MOFILE_DIR}/${GETTEXT_PACKAGE}.mo)

	# Merge .pot file changes to .po file
	ADD_CUSTOM_TARGET(msgmerge-${localeid}
		COMMAND ${XTC_MSGMERGE} -s -U ${POFILE} ${POTFILE}
		DEPENDS ${POTFILE} ${POFILE}
		)

	# If new locale id was added, add .po file creation routine
	IF(NOT EXISTS ${POFILE})
		ADD_CUSTOM_COMMAND(
			OUTPUT ${POFILE}
			COMMAND ${XTC_MSGINIT} -l ${localeid} -o ${POFILE} -i ${POTFILE}
			DEPENDS ${POTFILE}
			)
	ENDIF(NOT EXISTS ${POFILE})

	FILE(MAKE_DIRECTORY ${MOFILE_DIR})

	# Compile .po file to binary format .mo file
	ADD_CUSTOM_COMMAND(
		OUTPUT ${MOFILE}
		COMMAND ${XTC_MSGFMT} -c -v -o ${MOFILE} ${POFILE}
		DEPENDS ${POFILE}
		)

	# If translator program was given, add target to edit .po files with it
	IF(XTC_TRANSLATOR)
		ADD_CUSTOM_TARGET(msgtranslate-${localeid}
			COMMAND ${XTC_TRANSLATOR} ${POFILE}
			)
	ENDIF(XTC_TRANSLATOR)

	# Install all .mo files
	INSTALL(
		FILES ${MOFILE}
		DESTINATION ${XTRKCAD_LOCALEDIR}/${localeid}/LC_MESSAGES
		)

	SET(XTC_LOCALE_TARGETS
		${XTC_LOCALE_TARGETS}
		${i18n_BINARY_DIR}/${localeid}/LC_MESSAGES/xtrkcad.mo
		)

ENDFOREACH(localeid)

# Target to be executed with "make all". Xtrkcad must be built before this to
# make sure that the dynamically generated files messages.h and bllnhlp.c are
# up to date.
ADD_CUSTOM_TARGET(messages ALL
	DEPENDS xtrkcad ${POTFILE} ${XTC_LOCALE_TARGETS}
	)