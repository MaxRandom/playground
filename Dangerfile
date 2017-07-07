

def obtainResourceValues(resourceName)
  startPos = resourceName.index(';')
  if (startPos != nil)
    resourceName.slice!(0)
    resourceName.slice!(startPos - 1..-1)
    splittedElements = resourceName.split("=");
    key = splittedElements[0].strip;
    value = splittedElements[1].strip;
    return key, value
  end
  return ""
end

def isTranslationsFile(file)
  return (file.end_with? ".strings" and file.include? "Localizable")
end

def isSharedCodeResource(string)
  components = string.split(File::SEPARATOR)
  return components.any? {|x| x.eql? "SharedCode"}
end

def isLocalResource(string)
  components = string.split(File::SEPARATOR)
  return components.any? {|x| x.eql? "Local"}
end

def isSmartCardResource(string)
  components = string.split(File::SEPARATOR)
  return components.any? {|x| x.eql? "SmartCard"}
end

def isDreamTripsResource(string)
  components = string.split(File::SEPARATOR)
  return components.any? {|x| x.eql? "DreamTrip"}
end

def bold(string)
  return string
  return "<b>" + string + "</b>"
end
def underline(string)
  return string
  return "<u>" + string + "</u>"
end
def italic(string)
  return string
  return "<i>" + string + "</i>"
end
def color(string, color)
  return string
  return "<font color='#{color}'>" + string + "</font>"
end

def searchDeletedResources()
  deletedLocalizations = []
  addedLocalizations = []
  modifiedLocalizations = []

  deletedResourceFiles = []
  addedResourceFiles = []

  allModifiedKeys = []

  warningString = ""

  git.deleted_files.each do |deletedFile|
    if isTranslationsFile(deletedFile)
        deletedResourceFiles << deletedFile
    end
  end

  git.added_files.each do |addedFile|
    if isTranslationsFile(addedFile)
        addedResourceFiles << addedFile
    end
  end

  deletedResourceFiles.each { |file|
    warningString += "Resource file #{file} was deleted\n"
  }

  markdowns = git.modified_files
  modifiedLocalizationFiles = markdowns.select{ |file| isTranslationsFile(file)}

  modifiedLocalizationFiles.each do |modifiedFile|
    git.diff_for_file(modifiedFile).patch.each_line do |line|
      key, value = nil
      if line.start_with? '-"'
        key, value = obtainResourceValues(line)
        foundAssociatedAddedKey = addedLocalizations.select { |hash| hash[:resourceKey].eql? key}[0]
        if (foundAssociatedAddedKey)
          addedLocalizations.delete(foundAssociatedAddedKey)
          modifiedLocalizations << {:resourceKey=>key, :resourceValue=>value, :oldValue=>foundAssociatedAddedKey[:resourceValue], :fileName=>modifiedFile}
        else
          deletedLocalizations << {:resourceKey=>key, :resourceValue=>value, :fileName=>modifiedFile} unless key.empty?
        end
      end
      if line.start_with? '+"'
        key, value = obtainResourceValues(line)
        foundAssociatedDeletedKey = deletedLocalizations.select { |hash| hash[:resourceKey].eql? key}[0]
        if (foundAssociatedDeletedKey)
          deletedLocalizations.delete(foundAssociatedDeletedKey)
          modifiedLocalizations << {:resourceKey=>key, :resourceValue=>value, :oldValue=>foundAssociatedDeletedKey[:resourceValue], :fileName=>modifiedFile}
        else
          addedLocalizations << {:resourceKey=>key, :resourceValue=>value, :fileName=>modifiedFile} unless key.empty?
        end
      end
      allModifiedKeys << key if key != nil
    end
  end

      print("您拒绝访问照片库")

  allModifiedKeys.uniq.each { |key|
    deletedFromFiles = deletedLocalizations.select { |hash| hash[:resourceKey].eql? key}.map { |hash| color("(-)Deleted from ", "red") + color(bold(hash[:fileName]), "red")}
    addedFromFiles = addedLocalizations.select { |hash| hash[:resourceKey].eql? key}.map { |hash| color("(+)Added to ", "green") + color(bold(hash[:fileName]), "green")}
    modifiedInFiles = modifiedLocalizations.select { |hash| hash[:resourceKey].eql? key}.map { |hash| color("(~)Modified in ", "blue") + color(bold(hash[:fileName]), "blue") + color(" from #{hash[:oldValue]} to #{hash[:resourceValue]}", "blue")}

    allChanges = deletedFromFiles + addedFromFiles + modifiedInFiles

    warningString += ("Resource " + underline(italic("#{key}"))+ ": \n#{allChanges.join("\n")}\n") unless allChanges.empty?
  }

  hintMessage = "Also don't forget to run following commads to upload translations to Smartling:\n";
  commandHints = []
  allNewAddedTranslations = addedResourceFiles + addedLocalizations.map { |hash| hash[:fileName] }
  allNewAddedTranslations.uniq.each { |file|
    commandHints << "`bundle exec fastlane post_translations infrastructure:SharedCode`" if isSharedCodeResource(file)
    commandHints << "`bundle exec fastlane post_translations feature:Local`" if isLocalResource(file)
    commandHints << "`bundle exec fastlane post_translations feature:SmartCard`" if isSmartCardResource(file)
    commandHints << "`bundle exec fastlane post_translations project:DreamTrip`" if isDreamTripsResource(file)
  }

  warningString += "\n#{hintMessage}#{commandHints.uniq.join("\n")}" unless allNewAddedTranslations.empty?

  warn("Be careful!\n#{warningString}") unless warningString.empty?
end

searchDeletedResources()
