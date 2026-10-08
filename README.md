*This project has been created as part of the 42 curriculum by yhubaren.*

# get_next_line

## Description

**get_next_line** is a C function that reads a file descriptor one line at a time:

```c
char *get_next_line(int fd);
```

Each call returns the next line, including its terminating `\n` (except for the last line if the file does not end with one). When there is nothing left to read, or an error occurs, it returns `NULL`. The caller must free the returned string.

The project practices the `read()` system call, static variables, `malloc`/`free` and leak-free memory management.

## Instructions

Compile with the required flags and a buffer size:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c main.c
```

If `-D BUFFER_SIZE` is omitted, the header defines a default value of `42`. `main.c` is your own test program that calls `get_next_line()` in a loop and frees each line.

The project was also tested with a get_next_line tester, in addition to my own test program. Suggested tests: different `BUFFER_SIZE` values (`1`, `42`, `9999`, `10000000`), empty files, files with or without a final `\n`, very long lines, standard input, invalid file descriptors. Check memory with:

```bash
valgrind --leak-check=full ./a.out
```

## Algorithm

The function keeps a **static pointer `stash`** holding data that was read but not yet returned. Each call does three steps:

1. `read_file`: reads `BUFFER_SIZE` bytes at a time and appends them to `stash`, stopping as soon as `stash` contains a `\n` or `read()` returns `0`.
2. `extract_line`: copies everything up to and including the first `\n` into a new string, which is returned.
3. `clean_stash`: keeps only what follows that `\n` for the next call. If nothing is left, it frees the stash.

**Why this approach.** One `read()` can return several lines or only part of one, so the data after the first `\n` must survive between calls: this is why a static variable is needed. It works for any `BUFFER_SIZE`, reads only as much as needed (the whole file is never loaded at once), and uses no global variables. Old strings are freed whenever they are replaced, and on a `read()` or `malloc` error the allocated memory is freed and `NULL` is returned.

**Trade-off.** `ft_strjoin` copies the stash on every read, so very long lines with a tiny `BUFFER_SIZE` are slower than linear. This was accepted for simplicity.

## Resources

- [`read(2)` manual](https://man7.org/linux/man-pages/man2/read.2.html)
- [`malloc(3)` / `free(3)` manual](https://man7.org/linux/man-pages/man3/malloc.3.html)
- [Valgrind Memcheck manual](https://valgrind.org/docs/manual/mc-manual.html)

**AI usage:** AI  was used only for the documentation: checking this README against the subject and drafting its wording. The code was written by me.