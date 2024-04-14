# Creating shortcut keys using PERSONAL.XLSB

#### Category: MS Excel & VBA

---

### Ways that I use to help audiences process what they see on the screen a lot better

Ever since remote or hybrid models became a new norm in the workplace, it has been inevitable to share my screen for presenting my work or discussing ideas. The final or interim results of my work are typically in Excel; either the charts/tables summarizing data or the data itself that consists of rows and columns.

I would like to share a few things that I do at work to help people follow what you are presenting on Excel. This is not like presentation techniques or speech tips. So, anyone can implement and use it without any practice if they liked what I do.


### The big purple mouse cursor guy

Did somebody just call me?

![big-purple-mouse-cursor](images/big-purple-mouse-cursor.png)

I noticed that I keep moving my mouse pointer to where I am explaining. So, I came up with an idea to make my mouse pointer slightly bigger with distinct colours so people can easily find where I am. Go to mouse pointer (also called mouse properties) and customize it. Too much of anything is bad for you. I guess you don’t want to offend someone with a ridiculously huge cursor. It may make you look unprofessional if it is too big. Below is my current mouse pointer setting.

![mouse-pointer-option](images/mouse-pointer-option.png)


### Something for my own sake had unexpected benefits.

I used to occasionally lose my mouse cursor. It didn’t disappear, I was just not able to find it when I was working with multiple monitors. Then I found there was a setting that I can turn on to find it. The mouse properties also had a visibility aid under the pointer options tab.

Once turned on, a circle pops up to show you where the pointer is every time the control key is pressed. Issue resolved.

![mouse-properties](images/mouse-properties.png)

I also started to use this to draw attention to my screen and found it actually works well unless I use it too frequently.


### Main dish – Creating PERSONAL.XLSB to personalize shortcut keys

This is basically creating macros that can run on any workbook. Yes, any. You will record a macro stored in the “Personal Macro Workbook“. You are going to stop recording macro as soon as you started to record it. In most cases, macros stored in the workbook that it was recorded on can only be run on the same workbook. However, the macros recorded and stored in the personal macro workbook can be run on any Excel workbook.

![create-personal-macro](images/create-personal-macro.png)

Yes, I said stop recording as soon as it starts. Recording nothing? Sounds weird, isn’t it? This is just to create a hidden workbook called PERSONAL.XLSB that runs as long as Excel is opened. If you have never created any macro stored in the personal macro workbook before, this will create one. FYI, below is where the workbook is saved although you don’t necessarily need to know.

![personal-macro-xlsb](images/personal-macro-xlsb.png)

The type of the file is MS Excel Binary worksheet and the B in the file format XLSB stands for Binary. Some of you may already have had an experience where you were forced by Excel to save your work in XLSB format when the file became too big to be saved in regular XML-based .xlsx format. To make the explanation more straightforward, the same amount of data can be saved in a binary workbook much smaller than an ordinary Excel workbook in XML format. And it is capable of storing macros or VBA scripts. So, you have a file capable of storing and running macros that open as soon as Excel starts but is hidden and small enough so it won’t affect the performance of other Excel workbooks. These characteristics of the XLSB format are essential for macros that run on any Excel file.

Below are five colour changes to font or fill I frequently make. I always use colours in consistent ways so that people always know what it implies when I use a certain colour in my presentation.
– red_font: something negative. I use Ctrl + Shift + R
– green_font: something positive. I use Ctrl + Shift + G
– black_font: to change the colour back to neutral. I use Ctrl + Shift + B
– fill_yellow: manual input fields in templates. I use Ctrl + Shift + Y
– unfill: to erase the yellow. I use Ctrl + Shift + U

```
Sub red_font()
' Keyboard Shortcut: Ctrl+Shift+R
    With Selection.Font
        .Color = vbRed
        .TintAndShade = 0
    End With
End Sub

Sub green_font()
' Keyboard Shortcut: Ctrl+Shift+R
    With Selection.Font
        .Color = vbGreen
        .TintAndShade = 0
    End With
End Sub

Sub black_font()
' Keyboard Shortcut: Ctrl+Shift+B
    With Selection.Font
        .Color = vbBlack
        .TintAndShade = 0
    End With
End Sub

Sub fill_yellow()
' Keyboard Shortcut: Ctrl+Shift+Y
    With Selection.Interior
        .Pattern = xlSolid
        .Color = vbYellow
        .PatternTintAndShade = 0
    End With
End Sub

Sub Unfill()
' Keyboard Shortcut: Ctrl+Shift+U
    With Selection.Interior
        .Pattern = xlSolid
        .Pattern = xlNone
        .PatternTintAndShade = 0
    End With
End Sub
```

Open VBA editor (Alt + F11). Module 1 should be already created because you attempted to record something. Double-click on Module 1 and delete everything in it because it doesn’t really contain any macro when you immediately stopped recording.

![vba-editor](images/vba-editor.png)

Try copying the codes I shared above and paste them into the code section of Module 1.


### Not done yet – One last step to assign shortcut keys to each macro

Before you would be able to use shortcut keys for them, you need to assign shortcut keys to each macro.

First, you will need to make the PERSONAL.XLSB visible. Under the View ribbon, click Unhide and double-click on PERSONAL. This is because changing macro options such as adding the shortcut key is only allowed when the workbook where the macros are stored is visible.

![unhide-personal-macro-file](images/unhide-personal-macro-file.png)

You are now able to see the personal binary workbook window. The code pasted for five macros is all there when Macros under Developer are clicked on. Select one of them and click Options.

![choose-personal-macro](images/choose-personal-macro.png)

You can assign a key. I always try to use the key that is intuitive enough and aligns with the action. I thought B would be great for the black font. When I create my own shortcut keys in this way, I always try to avoid using Ctrl + [key] as a shortcut key because there could be default shortcut keys built in Excel and you don’t want your macros to overlap the pre-defined ones. For example, Ctrl + B is for making the font bolded. So, what I do is hold the Shift key until I see the “+Shift+” pops up, then press the desired key. Click ok then the macro black_font is going to be run when Ctrl+Shift+B is pressed. Finish repeating to assign shortcut keys to the rest of the macros.

![assign-shortcut](images/assign-shortcut.png)

Hide the personal binary workbook back. Otherwise, the workbook will be visible every time Excel is opened.

![hide-personal-macro-file](images/hide-personal-macro-file.png)

When you try to close Excel, it will ask you if you want to save the changes you made to the Personal workbook. Make sure you save them. You are ready to use the shortcut keys.

![save-personal-macro](images/save-personal-macro.png)

Although I found all of the above ideas helpful for explaining something when I share my screen remotely, I’ve been using them even before the pandemic started because looking at the same screen together in the office is screen-share at the end. I hope you find them useful and please feel free to contact me if you want to share your tips.
