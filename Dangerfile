

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
    def bold(string)
      return "<b>" + string + "</b>"
    end
    def underline(string)
      return "<u>" + string + "</u>"
    end
    def italic(string)
      return "<i>" + string + "</i>"
    end
    def color(string, color)
      return "<font color='#{color}'>" + string + "</font>"
    end

def searchDeletedResources()
      deletedLocalizations = []
      addedLocalizations = []
      modifiedLocalizations = []

      allModifiedKeys = []

      warningString = ""
      deletionPrefix = "-"

      git.deleted_files.each do |deletedFile|
        if isStringResourceFile(deletedFile)
          warningString += "Resource file #{deletedFile} was deleted\n"
        end
      end

      markdowns = git.modified_files
      modifiedLocalizationFiles = markdowns.select{ |file| file.end_with? ".strings" and file.include? "Localizable" }

      modifiedLocalizationFiles.each do |modifiedFile|
        git.diff_for_file(modifiedFile).patch.each_line do |line|
          key, value = nil
          if line.start_with? '-"'
            key, value = obtainResourceValues(line)
            unless key.empty?
              deletedLocalizations << {:resourceKey=>key, :resourceValue=>value, :fileName=>modifiedFile}
            end
          end
          if line.start_with? '+"'
            key, value = obtainResourceValues(line)
            foundAssociatedDeletedKey = deletedLocalizations.select { |hash| hash[:resourceKey].eql? key}[0]
            if (foundAssociatedDeletedKey)
              deletedLocalizations.delete(foundAssociatedDeletedKey)
              modifiedLocalizations << {:resourceKey=>key, :resourceValue=>value, :oldValue=>foundAssociatedDeletedKey[:resourceValue], :fileName=>modifiedFile}
            else
              addedLocalizations << {:resourceKey=>key, :resourceValue=>value, :fileName=>modifiedFile}
            end
          end
          allModifiedKeys << key if key != nil
        end
      end

        allModifiedKeys.uniq.each { |key|
        deletedFromFiles = deletedLocalizations.select { |hash| hash[:resourceKey].eql? key}.map { |hash| color("(-)Deleted from ", "red") + color(bold(hash[:fileName]), "red")}
        addedFromFiles = addedLocalizations.select { |hash| hash[:resourceKey].eql? key}.map { |hash| color("(+)Added to ", "green") + color(bold(hash[:fileName]), "green")}
        modifiedInFiles = modifiedLocalizations.select { |hash| hash[:resourceKey].eql? key}.map { |hash| color("(~)Modified in ", "blue") + color(bold(hash[:fileName]), "blue") + color(" from #{hash[:oldValue]} to #{hash[:resourceValue]}", "blue")}
        
        allChanges = deletedFromFiles + addedFromFiles + modifiedInFiles

        warningString += ("Resource " + underline(italic("#{key}"))+ ": \n#{allChanges.join("\n ")}\n\n") unless allChanges.empty?
      }

  warn("Be careful!\n#{warningString}") unless warningString.empty?
end

searchDeletedResources()
