# Datei-Sortierung/Zurücksetzungsskript
Das Ziel dieses Projekts ist es, ein Skript zu erstellen, welches Sachen auf einem Laptop automatisiert. Ich dachte, es wäre effektiv, ein Skript zu haben, das mir die Dateien sortiert, die die Erweiterungen .docx, .pptx und .xlsx in entsprechende Unterordner verschieben und auch die Sortierung zurücksetzen kann. Das zeigt an, dass das Skript die gewünschten Dateien erkennt und in einen bestimmten Unterordner sortiert oder die Sortierung zurücksetzt.
## Inhalt
Als Erstes habe ich Informationen gesucht, um überhaupt zu wissen, wie ich das mache. Habe ich ChatGPT verwendet, um herauszufinden, wie er es tun würde und seinen Skript studiert.

Der Befehl ```Move-Item``` hat mir bei diesem Projekt sehr geholfen, da ich mithilfe dieses Cmdlets die ausgewählten Dateien einfach in die entsprechenden Zielordner verschieben konnte. Dadurch konnte ich die Dateien effektiv sortieren und gegebenenfalls zusätzliche Sortierungskriterien hinzufügen.



```ps
 $files | ForEach-Object {
            $extension = $_.Extension
            $destinationFolder = Join-Path -Path $SourceFolderPath -ChildPath $extension
            Move-Item -Path $_.FullName -Destination $destinationFolder -ErrorAction Stop
            Write-Host "Datei $($_.Name) verschoben nach $destinationFolder"
            $movedFileCount++
        } 
```
Der Codeabschnitt stellt sicher, dass jede Datei in `$files` durchlaufen wird und unter Verwendung ihrer Erweiterung in den entsprechenden Zielordner verschoben wird.


## Was habe ich in diesem Auftrag gelernt?
Ich habe gelernt, wie man Dateien, mit Powershell Befehlen, verschiebt. Der Befehl lautet `Move-Item`. So verwendet man es:
```ps
 Move-Item -Path [Dateiname] -Destination [Zielort]
```
### Das Skriptt:
```ps
<#
.SYNOPSIS
    Dieses Skript sortiert Dateien nach ihren Erweiterungen in entsprechende Ordner oder macht die Sortierung rückgängig.

.DESCRIPTION
    Nach der Angabe eines Pfades fragt das Skript, ob Dateien sortiert oder eine bestehende Sortierung rückgängig gemacht werden soll.
    Bei der Sortierung werden Dateien mit den Erweiterungen .docx, .pptx und .xlsx in entsprechende Unterordner verschoben.
    Bei der Rückgängigmachung werden Dateien aus den Unterordnern in den Hauptordner verschoben und die Unterordner gelöscht.
    Zusätzlich wird in einer Log-Datei die Anzahl der verschobenen Dateien oder gelöschten Ordner verzeichnet.

.PARAMETER SourceFolderPath
    Der Pfad zum Ordner, der sortiert oder dessen Sortierung rückgängig gemacht werden soll.

.EXAMPLE
    .\FileSorter.ps1 -SourceFolderPath "C:\MeinOrdner"

.NOTES
    - PowerShell-Version: Mindestens 5.1 erforderlich.
    - Autor: Agachan Atputharasa
    - Datum: 1. Juli 2023
#>

$SourceFolderPath = Read-Host "Bitte geben Sie einen gültigen Pfad ein"

while (-not(Test-Path $SourceFolderPath -PathType 'Container')) {
    $SourceFolderPath = Read-Host "Bitte geben Sie einen gültigen Pfad ein"
}

$answer = Read-Host "Möchten Sie Dateien sortieren oder eine bestehende Sortierung rückgängig machen? (Sortieren/Rückgängig)"

while ($answer -ne "Sortieren" -and $answer -ne "Rückgängig") {
    $answer = Read-Host "Bitte antworten Sie mit 'Sortieren' oder 'Rückgängig'"
}

$LogFilePath = Join-Path -Path $SourceFolderPath -ChildPath "Log.txt"
$extensions = @(".docx", ".pptx", ".xlsx")

if ($answer -eq "Sortieren") {
    try {
        $movedFileCount = 0

        $extensions | ForEach-Object {
            $folderPath = Join-Path -Path $SourceFolderPath -ChildPath $_
            if (-not (Test-Path $folderPath -PathType 'Container')) {
                New-Item -Path $folderPath -ItemType 'Directory' | Out-Null
            }
        }

        $files = Get-ChildItem -Path $SourceFolderPath -File -Recurse |
        Where-Object { $_.Extension -in $extensions }

        $files | ForEach-Object {
            $extension = $_.Extension
            $destinationFolder = Join-Path -Path $SourceFolderPath -ChildPath $extension
            Move-Item -Path $_.FullName -Destination $destinationFolder -ErrorAction Stop
            Write-Host "Datei $($_.Name) verschoben nach $destinationFolder"
            $movedFileCount++
        }

        Add-Content -Path $LogFilePath -Value "Die Sortierung der Dateien wurde abgeschlossen. Insgesamt wurden $movedFileCount Dateien verschoben."

    }
    catch {
        Write-Host "Ein Fehler ist aufgetreten: $($_.Exception.Message)"
    }
}
elseif ($answer -eq "Rückgängig") {
    try {
        $deletedFolderCount = 0

        $extensions | ForEach-Object {
            $folderPath = Join-Path -Path $SourceFolderPath -ChildPath $_
            if (Test-Path $folderPath -PathType 'Container') {
                $files = Get-ChildItem -Path $folderPath -File
                if ($files.Count -gt 0) {
                    $files | ForEach-Object {
                        $destinationFolder = $SourceFolderPath
                        Move-Item -Path $_.FullName -Destination $destinationFolder -ErrorAction Stop
                        Write-Host "Datei $($_.Name) zurückverschoben nach $destinationFolder"
                    }
                }
                Remove-Item -Path $folderPath -Force -Recurse -ErrorAction Stop
                Write-Host "Ordner $folderPath gelöscht."
                $deletedFolderCount++
            }
        }

        Add-Content -Path $LogFilePath -Value "Die Rückgängigmachung der Dateisortierung wurde abgeschlossen. Insgesamt wurden $deletedFolderCount Ordner gelöscht."
    }
    catch {
        Write-Host "Ein Fehler ist aufgetreten: $($_.Exception.Message)"
    }
}

```

## Was das Programm zeigt
Das Programm zeigt eine Log Datei wo steht wie viele Dateien verschoben wurden bei der Sortierung und wie viele Ordner gelöscht wurden bei der Zurücksetzung der Sortierung.

## Selbstreflexion
Ich hatte am Anfang ein Problem, da ich nicht wusste, was ich machen sollte. Als ich dann aber eine Idee hatte, ging alles schnell, da ich ein konkretes Ziel vor Augen hatte. Während dem Programmieren hat mit GPT auch viel geholfen, obwohl es am Anfang ein bisschen zu viel war. Während dem Arbeiten konnte ich mich auch gut konzentrieren und war sehr wenig abgelenkt, vorallem im Vergleich zu vorherigen Projekten. 

## Reflexion
### Was habe ich gut gemacht?

-Diese Aufgabe war eine grossartige Gelegenheit für mich um meine bereits erworbenen Kenntnisse in C# aufzufrischen und anzuwenden. Es war motivierend zu sehen, wie ich die Fähigkeiten, die ich in einer Programmiersprache erworben hatte, in eine andere übertragen konnte, und dieser Prozess hat meine Fähigkeiten in beiden Sprachen gestärkt.

### Was habe ich weniger gut gemacht?

-Was nicht so gut war, war die Zeitmanagement. Ich war ignorant und dachte, dass ich das Projekt innerhalb einer Woche erledigem kann.Deshalb habe ich zu Beginn keine grossen Fortschritte gemacht und musste somit gegen Ende beeilen das Projekt abzuschliessen.

### Verbesserungsvorschlag
-Ich muss nächstes Mal ein bisschen früher mit den Aufgaben beginnen damit ich mich nicht beeilen muss.
