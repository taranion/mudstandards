---
sidebar_label: Pinkfish colour codes
---
# Pinkfish colour codes

*Pinkfish* (after its developer) have been used in some intermud protocols to support transmitting colour codes.

These are colour codes that do not require the (unprintable) ESC-character (like ANSI) and can thus be used over intermud lines and in remote who/finger answers.

These colours are not used in the CD library, but were developed by Pinkfish@Discworld to enable colours in players' say and tell commands. Some muds (like TMI-2) seem to support these codes in all commands, but most (like OuterSpace) only support Pinkfish codes for the intermud lines.

These codes are always surrounded by **%^** pairs. So in order to send the text *foobar* in red, use *%^RED%^foobar%^RESET%^*. If you want to quote the litteral text **%^**, it should be sent as **%%^^**.

Known codes are:

|         |           |           |
| ------- | --------- | --------- |
| RED     | B_RED     | FLASH     |
| BLUE    | B_BLUE    | BOLD      |
| ORANGE  | B_ORANGE  | REVERSE   |
| YELLOW  | B_YELLOW  | UNDERLINE |
| GREEN   | B_GREEN   | INITTERM  |
| BLACK   | B_BLACK   | RESET     |
| WHITE   | B_WHITE   | WINDOW    |
| CYAN    | B_CYAN    | ENDTERM   |
| MAGENTA | B_MAGENTA | STATUS    |

