# blender-scripts
A few scripts I used for the World War II Supercut (https://x.com/ww2supercut)

## Blender
### Find text across several selected clips
```python
for s in C.selected_sequences:
    if 'USA' in s.text:
        print(s.text)
```

### Set selected strip's text
```python
C.scene.sequence_editor.active_strip.text = 'Line 1\nLine 2'
```

### Print aspect ratio of video clip
```python
# Print Clip Info
sequence_strip = bpy.context.scene.sequence_editor.active_strip
video_width = sequence_strip.elements[0].orig_width
video_height = sequence_strip.elements[0].orig_height

# Calculate the aspect ratio
aspect_ratio = video_width / video_height
print(f"Width of the video clip: {video_width}")
print(f"Height of the video clip: {video_height}")
print(f"Aspect Ratio of the video clip: {aspect_ratio:.2f}")
```

### Render settings
```
* Set directory (Final slash important)
* Perceptually lossless
* Encoding speed good
* Set preview range to strips
* Set the range to preview range
* 29.97, 1080, stereo audio
* Set title animation, 1->1.3
* Set credits animation, 1 -> 0.8
* Final title date check and clip check, and final title card overlap
* Render Animation
* rename to # - NAME - raw
* Right click in the folder, open terminal here.
* Run through audio normalize:
ffmpeg -i '' -c:v copy -af loudnorm=I=-24:TP=-2:LRA=7:print_format=summary normalized.mp4
* Check volume metrics (printed as part of normalize script, should be about 25 and 2 for output integrated and peak)
* Rename with prefix
* Convert for Twitter:
ffmpeg -i '' -vf "scale=-1:720" -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k -movflags +faststart -pix_fmt yuv420p twitter.mp4
* Stitch in the srt file:
ffmpeg -i "$INPUT_VIDEO" -i "$INPUT_SUBTITLES" -c copy -c:s mov_text "$OUTPUT_VIDEO"
* Scan through it, make sure volume is good and subs line up
* Upload to google drive
* Upload to youtube
* Final QA watch with subtitles, log issues, only fix if p0
* Type up the Twitter post
* Upload to Twitter
* Rename srt to a .txt and add to post (otherwise get a cryptic error)
```

## FFMpeg

### Convert video for Twitter compatibility
```sh
ffmpeg -i ./Desktop/Trailer.mp4 -vf "scale=-1:720" -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k -movflags +faststart -pix_fmt yuv420p ./Desktop/Trailer_720p_fixed.mp4
```

### Pull mp3 file out of a video
```sh
ffmpeg -i VIDEOFILE -acodec libmp3lame -metadata TITLE="Name of Song" OUTPUTFILE.mp3
```

### Normalize audio (make loud sounds quieter and quiet sounds louder)
```sh
ffmpeg -i input_video.mp4 -c:v copy -af loudnorm=I=-24:TP=-2:LRA=7:print_format=summary output_video.mp4
```

### Add an srt file to an mp4
```sh
ffmpeg -i "$INPUT_VIDEO" -i "$INPUT_SUBTITLES" -c copy -c:s mov_text "$OUTPUT_VIDEO"
```

### Use AI to generate a subtitle file for a video (first install autosubtitle: https://github.com/m1guelpf/auto-subtitle)
```sh
auto_subtitle 'my_video.mp4' --srt_only True -o subtitled/ --language en --model large
```

## Subtitle Composer

### Custom Shortcuts
```
Shift+Left Decrease play rate
Shift+Right Increase play rate
Ctrl+J Join Lines
Ctrl+K Split Lines
CTRL+SHIFT+K custom script
Ctrl+Shift+Space seek to current line
Shift+Up/Down - Zoom waveform
Turn on spell checking, turn up autoscroll padding
```

### Script to split the current subtitle into multiple lines (Used in https://subtitlecomposer.kde.org/)
```
function splitStringIntoLines(inputString, maxLines) {
    var maxSingleLineLength = 25;
    var maxLineLength = 20;
    var tolerance = 3;

    if (inputString.length <= maxSingleLineLength) {
        return [inputString];
    }

    var words = inputString.split(' ');
    var lines = [];
    var currentLine = '';

    for (var i = 0; i < words.length; i++) {
        var word = words[i];
        if (currentLine.length + word.length + 1 <= maxSingleLineLength && lines.length === 0) {
            currentLine += (currentLine ? ' ' : '') + word;
        } else if (currentLine.length + word.length + 1 <= maxLineLength + tolerance) {
            currentLine += (currentLine ? ' ' : '') + word;
        } else {
            lines.push(currentLine);
            currentLine = word;
        }
    }

    if (currentLine) {
        lines.push(currentLine);
    }
    var pass = 0
    while (lines.length > maxLines && pass < 5) {
        var combinedLines = [];
        currentLine = '';

        for (var j = 0; j < lines.length; j++) {
            var line = lines[j];
            if (currentLine.length + line.length + 1 <= maxLineLength + tolerance) {
                currentLine += (currentLine ? ' ' : '') + line;
            } else {
                combinedLines.push(currentLine);
                currentLine = line;
            }
        }

        if (currentLine) {
            combinedLines.push(currentLine);
        }

        lines = combinedLines;
        pass += 1;
    }

    if (lines.length < maxLines) {
        var adjustedLines = [];
        currentLine = '';
        var avgLineLength = Math.ceil(inputString.length / maxLines);

        for (var k = 0; k < lines.length; k++) {
            var wordsInLine = lines[k].split(' ');

            for (var l = 0; l < wordsInLine.length; l++) {
                var word = wordsInLine[l];
                if (currentLine.length + word.length + 1 <= avgLineLength + tolerance) {
                    currentLine += (currentLine ? ' ' : '') + word;
                } else {
                    adjustedLines.push(currentLine);
                    currentLine = word;
                }
            }
        }

        if (currentLine) {
            adjustedLines.push(currentLine);
        }

        lines = adjustedLines;
    }

    return lines;
}

function printLines(lines) {
  for (var i = 0; i < lines.length; i++) {
    const line = lines[i];
    debug.information(lines[i]);
  }
}

// Example usage:
s = subtitle.instance();
const lineIndex = ranges.newSelectionRangeList().firstIndex()
const input = s.line(lineIndex).plainPrimaryText();
// debug.information(input);
const lines = splitStringIntoLines(input, 3);
//debug.information(lines)
// printLines(lines);
// debug.information(Object.keys(s.line(lineIndex)))
s.line(lineIndex).setPlainPrimaryText(lines.join('\n'))
```

## Handbrake

### Setup handbrake for ripping DVDs
```sh
sudo add-apt-repository ppa:stebbins/handbrake-releases
sudo apt-get update
sudo apt-get install handbrake
sudo apt-get install libdvd-pkg
sudo dpkg-reconfigure libdvd-pkg
sudo apt-get install build-essential pkg-config libc6-dev libssl-dev libexpat1-dev libavcodec-dev libgl1-mesa-dev qtbase5-dev zlib1g-dev
```
