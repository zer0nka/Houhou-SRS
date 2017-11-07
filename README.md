# Houhou SRS Project

## Overview

![Capture](http://houhou-srs.com/file/OV02.png)

**Houhou SRS** (or Houhou for short) is a Japanese kanji and vocabulary dictionary and learning software for Windows.

It works like any other English-Japanese dictionary, except that instead of forgetting the kanji and vocabulary that you are looking up, you can mark them for learning.

Kanji and vocabulary marked for learning will come back as "reviews", prompting you for the meanings and readings, in the hours, days, weeks, months after you marked them, as part of the SRS process, in order to make sure they get burned into your long-term memory.

The interface is designed to maximize time-efficiency in looking up and learning dictionary entries.

## Installation

### Installing the software (for users)
If you just want to install the software, [download the last version of the installer on the official website](http://houhou-srs.com/download/last).

Follow the directions of the installer and you'll be able to use it. [Microsoft .net Framework 4.5](http://www.microsoft.com/en-us/download/details.aspx?id=30653) is required, and will be installed with Houhou SRS if it was not already installed.

### Compiling the solution (for developers)
To be able to compile everything, after pulling, you will need some resources that are not part of the repository for multiple reasons (mainly because they are too heavy).
Here is a comprehensive list of these resources, along with an explanation of how to get them:
- Kanji.DatabaseMaker/Resources/**JMdict.xml**<br/>
   **A:** [Download the file from the edrdg website](http://ftp.monash.edu.au/pub/nihongo/JMdict.gz), unzip it, rename and use the unzipped file.
- Kanji.DatabaseMaker/Resources/**kanjidic2.xml**<br/>
   **A:** [Download the file from the project's website](http://www.csse.monash.edu.au/~jwb/kanjidic2/kanjidic2.xml.gz), unzip it and use the unzipped file.
- Kanji.Interface/Data/**KanjiDatabase.sqlite**<br/>
   **A:** This file is generated by the Kanji.DatabaseMaker project. Run that project and copy the output database file, or [download the file from Dropbox](https://db.tt/YiZkOLko).

### Compiling the installer
The installation script is located in the HouhouSetup directory in the repository. To compile it to an installer, you will need [inno setup](http://www.jrsoftware.org/isdl.php) (by Jordan Russel).
The installation script uses the Release binaries of the Interface project.

## Solution Architecture

### Overview
The solution is cut in several projects. There are library projects and executable projects.
The default startup project is Kanji.Interface (the main project containing views and the application logic).
We will go through all projects to explain what they are and how they are used.

- **Kanji.Common** is a *library project* referenced by all of the other Houhou projects. It contains essentially helpers and constants that are or could be required by other projects inside the solution.
- **Kanji.Database** is a *library project* that provides a database access logic layer for any projects making use of any of the two SQLite databases. It contains database files, DAOs and internal database access components.
- **Kanji.DatabaseMaker** is an *executable project* used to build the dictionary SQLite database file. It contains a set of ETL components that process open database resource files and build the SQLite database used by the *Kanji.Interface* project. This project should be run only when you need to generate a new dictionary database file.
- **Kanji.Interface** is an *executable project* containing Houhou's interface, and most of the application logic. Uses WPF with an MVVM structure.
- **NotifyIconWpf** is a *library project* created by Philipp Sumi and available on [CodeProject.com](http://www.codeproject.com/Articles/36468/WPF-NotifyIcon). It is a standalone library used by the *Kanji.Interface* project to implement the notification tray icon.

## Functional description

### Data
The project uses two **SQLite database files**. One is the dictionary database, and the other is the user database.

#### Dictionary database
The dictionary database is used by the Kanji.Interface project. It is stored as a resource of the project and required as an external resource file by the executable (Kanji.Interface/Data/KanjiDatabase.sqlite).
This database file can be generated by the Kanji.DatabaseMaker executable project.
It contains all **dictionary information** (kanji, radicals, words, definitions, word categories, etc), and should be used in **read-only** mode in the Kanji.Interface project. The dictionary database contains data from:
- [The JMdict project](http://www.edrdg.org/jmdict/j_jmdict.html);
- [The kanjidic2 file](http://www.csse.monash.edu.au/~jwb/kanjidic2/);
- [The radk/krad file](http://www.csse.monash.edu.au/~jwb/kradinf.html).

#### User database
The user database (or SRS database) is another SQLite database. An empty version is stored as part of the Kanji.Interface project (Kanji.Interface/Data/UserContent/SrsDatabase.sqlite) and is copied to the user resource directory used by Houhou (%USERPROFILE%/Documents/Houhou/SrsDatabase.sqlite).
That database is **user-specific** and contains the user's **SRS items** (as part of the SRS module of the Kanji.Interface project).

### Dictionary features
The dictionary features allow the user to look up for kanji and for words. Both kanji and words are stored in the dictionary database.

#### Kanji lookup
The kanji lookup allows you to retrieve a particular kanji from any information that you may have about it. You may know either one of its readings, its shape, one of its meanings, a word containing it, or a combination of any of these.

The kanji dictionary information is provided by the [kanjidic2 file](http://www.csse.monash.edu.au/~jwb/kanjidic2/).

**About the shape search, or "Kanji by radical" search:** While the implementation of the reading, meaning and containing word search is trivial, the shape search is a bit more unusual.
Thanks to the ["kradfile"](http://www.csse.monash.edu.au/~jwb/kradinf.html) resource, the dictionary database allows us to break down kanji characters in a set of sub-shapes, called radicals, that are usually part of several kanji characters. With this information, we are able to filter kanji by radicals selected visually by the user.

#### Words lookup
Vocabulary lookup is very trivial. Words can be searched either by reading (kanji or kana reading), meanings, by kanji, or any combination of these filters.

The word dictionary information is provided by the [JMdict project](http://www.edrdg.org/jmdict/j_jmdict.html).

### SRS features
The SRS (Spaced Repetition System) features constitute the "learning" part of Houhou SRS. When the user sees a kanji or a word in the dictionary and decides to learn it, they add it in their SRS item list.

Every item in the SRS item list has a level (starts at level 1) and a next review date. When the next review date is reached, the user is prompted for the item's meaning and reading. Answering both answers right will level up the item. Answering wrong to any answer will downgrade the item. The next review date is then scheduled with a delay matching the new level of the item. The higher the level is, the further the next review date will be.

### Tray icon and single-instance features
Houhou's interface comes with a notification tray icon. This icon is mainly used to display notifications about available reviews and works as follow:

At fixed time intervals, if reviews are available, the tray icon issues a notification bubble indicating the number of reviews available. Clicking the bubble will cause the main window to focus and to enter the review module.

The tray icon can remain active will the main window is closed. To properly exit the application, the user must right-click the tray icon and select "Exit" from the menu popping up.

When the window is closed and the user either tries to start another instance of the application or double-clicks the tray icon, the window shows up again. Note that only one instance of Houhou can be started at a time.

## Contributing
There are several ways to contribute to the project. Of course, you can help even if you're not a developer.

### Donations
I, Doublevil, author of the project, welcome your donations! If you enjoy Houhou and/or would like to support the project, please donate!

[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=L4L6EC2Y7C8QL&lc=US&item_name=Houhou%20SRS&currency_code=USD&bn=PP%2dDonationsBF%3abtn_donate_LG%2egif%3aNonHosted)

### Reports, requests, feedbacks
Something is wrong? Something is awesome? Something is missing? Please tell me what you think about Houhou! By reporting issues, submitting feature requests or simply giving feedback, you are contributing to the future of Houhou. So, if you have anything to say, please contact Doublevil at hello [at] houhou-srs.com.

### For developers
If you are a developer and would like to help, the best way to do that is to contribute to the source code. Why not fix some [known issues](https://github.com/Doublevil/Houhou-SRS/issues)? What about adding a feature you think the project lacks? Some performance optimization would be appreciated too!

The biggest thing to keep in mind is that the interface works with MVVM-style WPF. Other than that, I think you can get how things work by having a look at what's already there.

## Contact
If you have any questions or remarks regarding the project, don't hesitate to contact me (Doublevil) at hello [at] houhou-srs.com.
