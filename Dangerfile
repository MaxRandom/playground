markdowns = git.modified_files
modifiedLocalizationFiles = markdowns.select{ |file| file.end_with? ".strings" and file =~ "Localizable" }

modifiedLocalizationFiles.each{ |file| print(git.diff_for_file(file).patch)}
