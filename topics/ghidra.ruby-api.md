# Ghidra API (for Claude)

## Gotchas

- `java_import 'java.io.File'` shadows Ruby's `File` constant — use `IO.read` / `IO.write` instead of `File.read` / `File.write`
- `$current_api` is `com.goatshriek.rubydragon.ruby.RubyScript`, NOT `GhidraScript` — methods like `getProject()` don't exist

## Sample values

```
currentProgram.getName                               => Virus.Boot.PingPong.a
currentProgram.getExecutablePath                     => /path/to/original_project/Virus.Boot.PingPong.a  ← can be stale
getDomainFile.getName                                => Virus.Boot.PingPong.a
getDomainFile.getPathname                            => /Virus.Boot.PingPong.a  ← project-relative
getProjectLocator.getLocation                        => /path/to/project/  ← trailing slash
getProjectLocator.getName                            => Virus.Boot.PingPong.a
getProjectLocator.getProjectDir                      => /path/to/project/Virus.Boot.PingPong.a.rep
```

## $current_api (com.goatshriek.rubydragon.ruby.RubyScript)

```
addEntryPoint / add_entry_point
addInstructionXref / add_instruction_xref
addressFactory / address_factory
analyzeAll / analyze_all
analyzeChanges / analyze_changes
askAddress / ask_address
askBytes / ask_bytes
askChoice / ask_choice
askChoices / ask_choices
askDirectory / ask_directory
askDomainFile / ask_domain_file
askDouble / ask_double
askFile / ask_file
askInt / ask_int
askLanguage / ask_language
askLong / ask_long
askPassword / ask_password
askProgram / ask_program
askProjectFolder / ask_project_folder
askString / ask_string
askValues / ask_values
askYesNo / ask_yes_no
category
cleanup
clearBackgroundColor / clear_background_color
clearListing / clear_listing
closeProgram / close_program
codeUnitFormat / code_unit_format
createAddressSet / create_address_set
createAsciiString / create_ascii_string
createBookmark / create_bookmark
createByte / create_byte
createChar / create_char
createClass / create_class
createDWord / create_dword
createData / create_data
createDouble / create_double
createDwords / create_dwords
createEquate / create_equate
createExternalReference / create_external_reference
createFloat / create_float
createFragment / create_fragment
createFunction / create_function
createHighlight / create_highlight
createLabel / create_label
createMemoryBlock / create_memory_block
createMemoryReference / create_memory_reference
createNamespace / create_namespace
createProgram / create_program
createQWord / create_qword
createSelection / create_selection
createStackReference / create_stack_reference
createSymbol / create_symbol
createTableChooserDialog / create_table_chooser_dialog
createUnicodeString / create_unicode_string
createWord / create_word
currentHighlight= / current_highlight=
currentLocation= / current_location=
currentProgram / current_program
currentSelection= / current_selection=
disassemble
findBytes / find_bytes
findPascalStrings / find_pascal_strings
findStrings / find_strings
firstData / first_data
firstFunction / first_function
firstInstruction / first_instruction
getAddressFactory / get_address_factory
getAnalysisOptionDefaultValue / get_analysis_option_default_value
getAnalysisOptionDefaultValues / get_analysis_option_default_values
getAnalysisOptionDescription / get_analysis_option_description
getAnalysisOptionDescriptions / get_analysis_option_descriptions
getBookmarks / get_bookmarks
getByte / get_byte
getBytes / get_bytes
getCategory / get_category
getCodeUnitFormat / get_code_unit_format
getControls / get_controls
getCurrentAnalysisOptionsAndValues / get_current_analysis_options_and_values
getCurrentProgram / get_current_program
getDataAfter / get_data_after
getDataAt / get_data_at
getDataBefore / get_data_before
getDataContaining / get_data_containing
getDataTypes / get_data_types
getDefaultLanguage / get_default_language
getDemangled / get_demangled
getDouble / get_double
getEOLComment / get_eol_comment
getEOLCommentAsRendered / get_eol_comment_as_rendered
getEquate / get_equate
getEquates / get_equates
getFirstData / get_first_data
getFirstFunction / get_first_function
getFirstInstruction / get_first_instruction
getFloat / get_float
getFragment / get_fragment
getFunction / get_function
getFunctionAfter / get_function_after
getFunctionAt / get_function_at
getFunctionBefore / get_function_before
getFunctionContaining / get_function_containing
getGhidraVersion / get_ghidra_version
getGlobalFunctions / get_global_functions
getInstructionAfter / get_instruction_after
getInstructionAt / get_instruction_at
getInstructionBefore / get_instruction_before
getInstructionContaining / get_instruction_containing
getInt / get_int
getLanguage / get_language
getLastData / get_last_data
getLastFunction / get_last_function
getLastInstruction / get_last_instruction
getLong / get_long
getMemoryBlock / get_memory_block
getMemoryBlocks / get_memory_blocks
getMonitor / get_monitor
getNamespace / get_namespace
getPlateComment / get_plate_comment
getPlateCommentAsRendered / get_plate_comment_as_rendered
getPostComment / get_post_comment
getPostCommentAsRendered / get_post_comment_as_rendered
getPreComment / get_pre_comment
getPreCommentAsRendered / get_pre_comment_as_rendered
getProgramFile / get_program_file
getProjectRootFolder / get_project_root_folder
getReference / get_reference
getReferencesFrom / get_references_from
getReferencesTo / get_references_to
getRepeatableComment / get_repeatable_comment
getRepeatableCommentAsRendered / get_repeatable_comment_as_rendered
getReusePreviousChoices / get_reuse_previous_choices
getScriptAnalysisMode / get_script_analysis_mode
getScriptArgs / get_script_args
getScriptName / get_script_name
getShort / get_short
getSourceFile / get_source_file
getState / get_state
getSymbol / get_symbol
getSymbolAfter / get_symbol_after
getSymbolAt / get_symbol_at
getSymbolBefore / get_symbol_before
getSymbols / get_symbols
getUndefinedDataAfter / get_undefined_data_after
getUndefinedDataAt / get_undefined_data_at
getUndefinedDataBefore / get_undefined_data_before
getUserName / get_user_name
ghidraVersion / ghidra_version
goTo / go_to
importFile / import_file
importFileAsBinary / import_file_as_binary
isAnalysisOptionDefaultValue / is_analysis_option_default_value
isRunningHeadless / is_running_headless
lastData / last_data
lastFunction / last_function
lastInstruction / last_instruction
loadPropertiesFile / load_properties_file
loadVariablesFromState / load_variables_from_state
memoryBlocks / memory_blocks
monitor
openDataTypeArchive / open_data_type_archive
openProgram / open_program
parseAddress / parse_address
parseBoolean / parse_boolean
parseBytes / parse_bytes
parseChoice / parse_choice
parseChoices / parse_choices
parseDirectory / parse_directory
parseDomainFile / parse_domain_file
parseDouble / parse_double
parseInt / parse_int
parseLanguageCompileSpecPair / parse_language_compile_spec_pair
parseLong / parse_long
parseProjectFolder / parse_project_folder
popup
print / printerr / printf / println
programFile / program_file
projectRootFolder / project_root_folder
removeBookmark / remove_bookmark
removeData / remove_data
removeDataAt / remove_data_at
removeEntryPoint / remove_entry_point
removeEquate / remove_equate
removeEquates / remove_equates
removeFunction / remove_function
removeFunctionAt / remove_function_at
removeHighlight / remove_highlight
removeInstruction / remove_instruction
removeInstructionAt / remove_instruction_at
removeMemoryBlock / remove_memory_block
removeReference / remove_reference
removeSelection / remove_selection
removeSymbol / remove_symbol
resetAllAnalysisOptions / reset_all_analysis_options
resetAnalysisOption / reset_analysis_option
resetAnalysisOptions / reset_analysis_options
reusePreviousChoices / reuse_previous_choices
runCommand / run_command
runScript / run_script
runScriptPreserveMyState / run_script_preserve_my_state
runningHeadless / running_headless
saveProgram / save_program
scriptAnalysisMode / script_analysis_mode
scriptArgs / script_args
scriptName / script_name
setAnalysisOption / set_analysis_option
setAnalysisOptions / set_analysis_options
setAnonymousServerCredentials / set_anonymous_server_credentials
setBackgroundColor / set_background_color
setByte / set_byte
setBytes / set_bytes
setCurrentHighlight / set_current_highlight
setCurrentLocation / set_current_location
setCurrentSelection / set_current_selection
setDouble / set_double
setEOLComment / set_eol_comment
setFloat / set_float
setInt / set_int
setLong / set_long
setPlateComment / set_plate_comment
setPostComment / set_post_comment
setPotentialPropertiesFileLocations / set_potential_properties_file_locations
setPreComment / set_pre_comment
setPropertiesFile / set_properties_file
setPropertiesFileLocation / set_properties_file_location
setReferencePrimary / set_reference_primary
setRepeatableComment / set_repeatable_comment
setReusePreviousChoices / set_reuse_previous_choices
setScriptArgs / set_script_args
setServerCredentials / set_server_credentials
setShort / set_short
setSourceFile / set_source_file
setToolStatusMessage / set_tool_status_message
show
sourceFile / source_file
state
toAddr / to_addr
toHexString / to_hex_string
updateStateFromVariables / update_state_from_variables
userName / user_name
```

## currentProgram (ghidra.program.database.ProgramDB)

```
addCloseListener / add_close_listener
addConsumer / add_consumer
addDomainFileListener / add_domain_file_listener
addListener / add_listener
addSynchronizedDomainObject / add_synchronized_domain_object
addTransactionListener / add_transaction_listener
addressFactory / address_factory
addressMap / address_map
allRedoNames / all_redo_names
allUndoNames / all_undo_names
associatedUserFilesystem / associated_user_filesystem
bookmarkManager / bookmark_manager
canLock / can_lock
canRedo / can_redo
canSave / can_save
canUndo / can_undo
changeSet / change_set
changeStatus / change_status
changeable
changed
checkExclusiveAccess / check_exclusive_access
clearCache / clear_cache
clearUndo / clear_undo
close
codeManager / code_manager
compiler
compilerSpec / compiler_spec
consumerList / consumer_list
createAddressSetPropertyMap / create_address_set_property_map
createIntRangeMap / create_int_range_map
createOverlaySpace / create_overlay_space
createPrivateEventQueue / create_private_event_queue
creationDate / creation_date
currentTransactionInfo / current_transaction_info
dBHandle / db_handle
dataTypeManager / data_type_manager
defaultPointerSize / default_pointer_size
deleteAddressRange / delete_address_range
deleteAddressSetPropertyMap / delete_address_set_property_map
deleteIntRangeMap / delete_int_range_map
description
domainFile / domain_file
effectiveImageBase= / effective_image_base=
endTransaction / end_transaction
equateTable / equate_table
eventsEnabled= / events_enabled=
executableFormat / executable_format
executableMD5 / executable_md5
executablePath / executable_path        ← import path (can be stale)
executableSHA256 / executable_sha256
externalManager / external_manager
fatalErrorOccurred / fatal_error_occurred
fireEvent / fire_event
flushEvents / flush_events
flushPrivateEventQueue / flush_private_event_queue
flushWriteCache / flush_write_cache
forceLock / force_lock
functionManager / function_manager
getAddressFactory / get_address_factory
getAddressMap / get_address_map
getAddressSetPropertyMap / get_address_set_property_map
getAllRedoNames / get_all_redo_names
getAllUndoNames / get_all_undo_names
getAssociatedUserFilesystem / get_associated_user_filesystem
getBookmarkManager / get_bookmark_manager
getChangeSet / get_change_set
getChangeStatus / get_change_status
getChanges / get_changes
getCodeManager / get_code_manager
getCompiler / get_compiler
getCompilerSpec / get_compiler_spec
getConsumerList / get_consumer_list
getCreationDate / get_creation_date
getCurrentTransactionInfo / get_current_transaction_info
getDBHandle / get_db_handle
getDataTypeManager / get_data_type_manager
getDefaultPointerSize / get_default_pointer_size
getDescription / get_description
getDomainFile / get_domain_file
getEquateTable / get_equate_table
getExecutableFormat / get_executable_format
getExecutableMD5 / get_executable_md5
getExecutablePath / get_executable_path  ← import path (can be stale)
getExecutableSHA256 / get_executable_sha256
getExternalManager / get_external_manager
getFunctionManager / get_function_manager
getGlobalNamespace / get_global_namespace
getImageBase / get_image_base
getIntRangeMap / get_int_range_map
getLanguage / get_language
getLanguageCompilerSpecPair / get_language_compiler_spec_pair
getLanguageID / get_language_id
getListing / get_listing
getLock / get_lock
getMaxAddress / get_max_address
getMemory / get_memory
getMetadata / get_metadata
getMinAddress / get_min_address
getModificationNumber / get_modification_number
getName / get_name
getNamespaceManager / get_namespace_manager
getOptions / get_options
getOptionsNames / get_options_names
getPreferredRootNamespaceCategoryPath / get_preferred_root_namespace_category_path
getProgramContext / get_program_context
getProgramUserData / get_program_user_data
getRedoName / get_redo_name
getReferenceManager / get_reference_manager
getRegister / get_register
getRegisters / get_registers
getRelocationTable / get_relocation_table
getSourceFileManager / get_source_file_manager
getStoredVersion / get_stored_version
getSymbolTable / get_symbol_table
getSynchronizedDomainObjects / get_synchronized_domain_objects
getTreeManager / get_tree_manager
getUndoName / get_undo_name
getUndoStackDepth / get_undo_stack_depth
getUniqueProgramID / get_unique_program_id
getUserData / get_user_data
getUsrPropertyManager / get_usr_property_manager
globalNamespace / global_namespace
hasExclusiveAccess / has_exclusive_access
hasTerminatedTransaction / has_terminated_transaction
imageBase / image_base
installExtensions / install_extensions
invalidate
invalidateWriteCache / invalidate_write_cache
isChangeable / is_changeable
isChanged / is_changed
isClosed / is_closed
isLanguageUpgradePending / is_language_upgrade_pending
isLocked / is_locked
isSendingEvents / is_sending_events
isTemporary / is_temporary
isUsedBy / is_used_by
language
languageCompilerSpecPair / language_compiler_spec_pair
languageID / language_id
languageUpgradePending / language_upgrade_pending
listing
loadMetadata / load_metadata
lock
maxAddress / max_address
memory
metadata
minAddress / min_address
modificationNumber / modification_number
moveAddressRange / move_address_range
name
namespaceManager / namespace_manager
openTransaction / open_transaction
optionsNames / options_names
parseAddress / parse_address
performPropertyListAlterations / perform_property_list_alterations
preferredRootNamespaceCategoryPath / preferred_root_namespace_category_path
programContext / program_context
programUserData / program_user_data
redo
redoName / redo_name
referenceManager / reference_manager
release
releaseSynchronizedDomainObject / release_synchronized_domain_object
relocationTable / relocation_table
removeCloseListener / remove_close_listener
removeDomainFileListener / remove_domain_file_listener
removeListener / remove_listener
removeOverlaySpace / remove_overlay_space
removePrivateEventQueue / remove_private_event_queue
removeTransactionListener / remove_transaction_listener
renameOverlaySpace / rename_overlay_space
restoreImageBase / restore_image_base
save
saveMetadata / save_metadata
saveToPackedFile / save_to_packed_file
sendingEvents / sending_events
setChanged / set_changed
setCompiler / set_compiler
setDomainFile / set_domain_file
setEffectiveImageBase / set_effective_image_base
setEventsEnabled / set_events_enabled
setExecutableFormat / set_executable_format
setExecutableMD5 / set_executable_md5
setExecutablePath / set_executable_path
setExecutableSHA256 / set_executable_sha256
setImageBase / set_image_base
setImmutable / set_immutable
setLanguage / set_language
setName / set_name
setObjChanged / set_obj_changed
setPreferredRootNamespaceCategoryPath / set_preferred_root_namespace_category_path
setPropertyChanged / set_property_changed
setPropertyRangeRemoved / set_property_range_removed
setRegisterValuesChanged / set_register_values_changed
setTemporary / set_temporary
sourceFileManager / source_file_manager
startTransaction / start_transaction
storedVersion / stored_version
symbolTable / symbol_table
synchronizedDomainObjects / synchronized_domain_objects
treeManager / tree_manager
undo
undoName / undo_name
undoStackDepth / undo_stack_depth
uniqueProgramID / unique_program_id
unlock
updateMetadata / update_metadata
usedBy / used_by
userData / user_data
usrPropertyManager / usr_property_manager
withTransaction / with_transaction
```

## currentProgram.getDomainFile (ghidra.framework.data.GhidraFile)

```
absoluteLinkPath / absolute_link_path
addToVersionControl / add_to_version_control
busy
canAddToRepository / can_add_to_repository
canCheckin / can_checkin
canCheckout / can_checkout
canMerge / can_merge
canRecover / can_recover
canSave / can_save
changed
changesByOthersSinceCheckout / changes_by_others_since_checkout
checkedOut / checked_out
checkedOutExclusive / checked_out_exclusive
checkin
checkout
checkoutStatus / checkout_status
checkouts
compareTo / compare_to
consumers
contentType / content_type
copyTo / copy_to
copyToAsLink / copy_to_as_link
copyVersionTo / copy_version_to
delete
domainObjectClass / domain_object_class
exists
externalLink / external_link
file
fileID / file_id
folderLink / folder_link
getAbsoluteLinkPath / get_absolute_link_path
getChangesByOthersSinceCheckout / get_changes_by_others_since_checkout
getCheckoutStatus / get_checkout_status
getCheckouts / get_checkouts
getConsumers / get_consumers
getContentType / get_content_type
getDomainObject / get_domain_object
getDomainObjectClass / get_domain_object_class
getFile / get_file
getFileID / get_file_id
getIcon / get_icon
getImmutableDomainObject / get_immutable_domain_object
getLastModifiedTime / get_last_modified_time
getLatestVersion / get_latest_version
getLinkInfo / get_link_info
getLinkPath / get_link_path
getLinkStatus / get_link_status
getLinkedFolder / get_linked_folder
getLocalProjectURL / get_local_project_url
getMetadata / get_metadata
getName / get_name
getOpenedDomainObject / get_opened_domain_object
getParent / get_parent
getPathname / get_pathname           ← project-relative path
getProjectLocator / get_project_locator   ← use this for filesystem path
getReadOnlyDomainObject / get_read_only_domain_object
getSharedProjectURL / get_shared_project_url
getUserFileSystem / get_user_file_system
getVersion / get_version
getVersionHistory / get_version_history
hijacked
inWritableProject / in_writable_project
isBusy / is_busy
isChanged / is_changed
isCheckedOut / is_checked_out
isCheckedOutExclusive / is_checked_out_exclusive
isExternalLink / is_external_link
isFolderLink / is_folder_link
isHijacked / is_hijacked
isInWritableProject / is_in_writable_project
isLatestVersion / is_latest_version
isLink / is_link
isLinkingSupported / is_linking_supported
isOpen / is_open
isReadOnly / is_read_only
isVersioned / is_versioned
lastModifiedTime / last_modified_time
latestVersion / latest_version
length
link
linkInfo / link_info
linkPath / link_path
linkedFolder / linked_folder
linkingSupported / linking_supported
merge
metadata
modifiedSinceCheckout / modified_since_checkout
moveTo / move_to
name
open
packFile / pack_file
parent
pathname
projectLocator / project_locator     ← use this for filesystem path
readOnly / read_only
save
setName / set_name
setReadOnly / set_read_only
takeRecoverySnapshot / take_recovery_snapshot
terminateCheckout / terminate_checkout
undoCheckout / undo_checkout
userFileSystem / user_file_system
version
versionHistory / version_history
versioned
```

## currentProgram.getDomainFile.getProjectLocator (ghidra.framework.model.ProjectLocator)

```
exists
getLocation / get_location    ← filesystem dir of .gpr file, WITH trailing slash
getMarkerFile / get_marker_file
getName / get_name            ← project name (= program name)
getProjectDir / get_project_dir   ← <name>.rep directory
getProjectLockFile / get_project_lock_file
getURL / get_url
isTransient / is_transient
location                      ← same as getLocation
markerFile / marker_file
name                          ← same as getName
projectDir / project_dir      ← same as getProjectDir
projectLockFile / project_lock_file
url                           ← same as getURL
```
