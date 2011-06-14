# Easy note taking in Vim

The notes.vim plug-in for the [Vim text editor] [vim] makes it easy to manage your notes in Vim:

 * **Starting a new note:** Execute the `:Note` command to create a new buffer and load the appropriate file type and syntax
   * You can also start a new note with the selected text as title using the `:NoteFromSelectedText` command
 * **Saving notes:** Just use Vim's [:write] [write] and [:update] [update] commands, you don't need to provide a filename because it will be set based on the title (first line) of your note (you also don't need to worry about special characters, they'll be escaped)
 * **Editing existing notes:** Execute `:Note anything` to edit a note containing `anything` in its title (if no notes are found a new one is created with its title set to `anything`)
 * **Deleting notes:** The `:DeleteNote` command enables you to delete the current note
 * **Searching notes:** `:SearchNotes keyword …` searches for keywords and `:SearchNotes /pattern/` searches for regular expressions
   * **Smart defaults:** Without an argument `:SearchNotes` searches for the word under the cursor (if the word starts with `@` that character will be included in the search, this means you can easily search for *@tagged* notes)
   * **Back-references:** The `:RelatedNotes` command find all notes referencing the current file
   * A [Python 2] [python] script is included that accelerates keyword searches using an [SQLite] [sqlite] database
   * The `:RecentNotes` command lists your notes by modification date, starting with the most recently edited note
 * **Navigating between notes:** The included file type plug-in redefines [gf] [gf] to jump between notes and the syntax script highlights note names as hyper links
 * **Writing aids:** The included file type plug-in contains mappings for automatic curly quotes, arrows and list bullets and supports completion of note titles using Control-X Control-U
 * **Embedded file types:** The included syntax script supports embedded highlighting using blocks marked with `{{{type … }}}` which allows you to embed highlighted code and configuration snippets in your notes

Here's a screen shot of the syntax mode using the [slate] [slate] color scheme:

![Syntax mode screen shot](http://peterodding.com/code/vim/notes/syntax.png)

## Install & usage

Unzip the most recent [ZIP archive] [download] file inside your Vim profile directory (usually this is `~/.vim` on UNIX and `%USERPROFILE%\vimfiles` on Windows), restart Vim and execute the command `:helptags ~/.vim/doc` (use `:helptags ~\vimfiles\doc` instead on Windows). To get started execute `:Note` or `:edit note:`, this will start a new note that contains instructions on how to continue from there (and how to use the plug-in in general).

## Options

All options have reasonable defaults so if the plug-in works after installation you don't need to change any options. They're available for people who like to customize their directory layout. These options can be configured in your [vimrc] [vimrc] by including a line like this:

    let g:notes_directory = '~/Documents/Notes'

### The `g:notes_directory` option

All your notes are stored together in one directory. This option defines the path of this directory.

### The `g:notes_suffix` option

The suffix to add to generated filenames. The plug-in generates filenames for your notes based on the title (first line) of each note and by default these filenames don't include an extension like `.txt`. You can use this option to make the plug-in automatically append an extension without having to embed the extension in the note's title, e.g.:

    :let g:notes_suffix = '.txt'

### The `g:notes_shadowdir` option

The notes plug-in comes with some default notes containing documentation about the plug-in. This option defines the path of the directory containing these notes.

### The `g:notes_indexfile` option

This option defines the pathname of the optional keyword index used by the plug-in to perform accelerated keyword searching.

### The `g:notes_indexscript` option

This option defines the pathname of the Python script that's used to perform accelerated keyword searching.

## Commands

To edit one of your existing notes you can use Vim commands such as [:edit] [edit], [:split] [split] and [:tabedit] [tabedit] with a filename that starts with *note:* followed by (part of) the title of one of your notes, e.g.:

    :edit note:todo

This shortcut also works from the command line:

    $ gvim note:todo

When you don't follow *note:* with anything a new note is created.

### The `:Note` command

When executed without any arguments this command starts a new note in the current window. If you pass one or more arguments the command will edit an existing note containing the given words in the title. If more than one note is found you'll be asked which note you want to edit. If no notes are found a new note is started with the given word(s) as title.

This command will fail when changes have been made to the current buffer, unless you use `:Note!` which discards any changes.

*This command supports tab completion:* If you complete one word, all existing notes containing the given word somewhere in their title are suggested. If you type more than one word separated by spaces, the plug-in will complete only the missing words so that the resulting command line contains the complete note title and nothing more.

### The `:NoteFromSelectedText` command

When you execute this command it will start a new note with the selected text as the title of the note.

### The `:DeleteNote` command

The `:DeleteNote` command deletes the current note, destroys the buffer and removes the note from the internal cache of filenames and note titles. This fails when changes have been made to the current buffer, unless you use `:DeleteNote!` which discards any changes.

### The `:SearchNotes` command

This command wraps [:vimgrep] [vimgrep] and enables you to search through your notes using one or more keywords or a regular expression pattern. To search for a pattern you pass a single argument that starts/ends with a slash:

    :SearchNotes /TODO\|FIXME\|XXX/

To search for one or more keywords you can just omit the slashes, this matches notes containing all of the given keywords:

    :SearchNotes syntax highlighting

#### `:SearchNotes` understands @tags

If you don't pass any arguments to the `:SearchNotes` command it will search for the word under the cursor. If the word under the cursor starts with '@' this character will be included in the search, which makes it possible to easily add *@tags* to your *@notes* and then search for those tags. To make searching for tags even easier you can create key mappings for the `:SearchNotes` command:


    " Make the C-] combination search for @tags:
    imap <C-]> <C-o>:SearchNotes<CR>
    nmap <C-]> :SearchNotes<CR>

    " Make double mouse click search for @tags. This is actually quite a lot of
    " fun if you don't use the mouse for text selections anyway; you can click
    " between notes as if you're in a web browser:
    imap <2-LeftMouse> <C-o>:SearchNotes<CR>
    nmap <2-LeftMouse> :SearchNotes<CR>

These mappings are currently not enabled by default because they conflict with already useful key mappings, but if you have any suggestions for alternatives feel free to contact me through GitHub or at <peter@peterodding.com>.

#### Accelerated searching with Python and SQLite

After collecting a fair amount of notes (say more than 5 MB) you will probably start to get annoyed at how long it takes Vim to search through all of your notes. To make searching more scalable the notes plug-in includes a Python script which uses a full text index of your notes stored in an SQLite database.

The first time the Python script is run it will need to build the complete index which can take a few minutes, but after the index has been initialized updates and searches should be more or less instantaneous.

### The `:RelatedNotes` command

This command makes it easy to find all notes related to the current file: If you are currently editing a note then a search for the note's title is done, otherwise this searches for the absolute path of the current file.

### The `:RecentNotes` command

If you execute the `:RecentNotes` command it will open a Vim buffer that lists all your notes grouped by the day they were edited, starting with your most recently edited note. If you pass an argument to `:RecentNotes` it will filter the list of notes by matching the title of each note against the argument which is interpreted as a Vim pattern.

## Contact

If you have questions, bug reports, suggestions, etc. the author can be contacted at <peter@peterodding.com>. The latest version is available at <http://peterodding.com/code/vim/notes/> and <http://github.com/xolox/vim-notes>. If you like the script please vote for it on [Vim Online] [vim_online].

## License

This software is licensed under the [MIT license] [mit].  
© 2011 Peter Odding &lt;<peter@peterodding.com>&gt;.

[vim]: http://www.vim.org/
[write]: http://vimdoc.sourceforge.net/htmldoc/editing.html#:write
[update]: http://vimdoc.sourceforge.net/htmldoc/editing.html#:update
[python]: http://python.org/
[sqlite]: http://sqlite.org/
[gf]: http://vimdoc.sourceforge.net/htmldoc/editing.html#gf
[slate]: http://code.google.com/p/vim/source/browse/runtime/colors/slate.vim
[download]: http://peterodding.com/code/vim/downloads/notes.zip
[vimrc]: http://vimdoc.sourceforge.net/htmldoc/starting.html#vimrc
[edit]: http://vimdoc.sourceforge.net/htmldoc/editing.html#:edit
[split]: http://vimdoc.sourceforge.net/htmldoc/windows.html#:split
[tabedit]: http://vimdoc.sourceforge.net/htmldoc/tabpage.html#:tabedit
[vimgrep]: http://vimdoc.sourceforge.net/htmldoc/quickfix.html#:vimgrep
[vim_online]: http://www.vim.org/scripts/script.php?script_id=3375
[mit]: http://en.wikipedia.org/wiki/MIT_License
