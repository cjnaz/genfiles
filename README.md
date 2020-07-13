# genfiles

genfiles creates a mass of files in a directory hierarchy for testing purposes.  I have used this tool for generating 20,000 files over 100 directories for test cases for rclonesync.

The primary switches are...
- the number of files to generate
- distributed over a number of directories
- at a specified directory depth

The files created may be copies of an existing file, or may be empty (default).  


## Usage
```
$ ./genfiles -h
usage: genfiles [-h] [-f NUM_FILES] [-d NUM_DIRS] [-e DIRS_DEPTH]
                [--dir-node-base DIR_NODE_BASE]
                [--dir-leaf-base DIR_LEAF_BASE] [--file-base FILE_BASE]
                [--file-content FILE_CONTENT] [-n] [-v] [-V]
                RootPath

Generate a tree of files for testing purposes.

The total number of num-files are evenly distributed over num-dirs, where possible.
The total number of num-dirs are distributed over 2**(dirs-depth -1) bottom node-level directories.
The leaf dirs are effectively at the dirs-depth level.
The precise number of files are created; however, the last leaf dir will have fewer files if 
the num-files/num-dirs ratio is not a whole number.
Fewer than num-dirs are created if the dirs-depth and num-files/num-dirs ratios don't result
in a whole number.

EG:
    "--num-file 100 --num-dirs 20 --dirs-depth 3"
  generates
    gentree/node_A/node_A/leaf_1 (containing file_1 to file_5)
    ...
    gentree/node_B/node_B/leaf_20 (containing file_96 to file_100)

positional arguments:
  RootPath              Dir must exist.  <gentree/> created here containing generated hierarchy.

optional arguments:
  -h, --help            show this help message and exit
  -f NUM_FILES, --num-files NUM_FILES
                        Number of files to generate (default = 100).
  -d NUM_DIRS, --num-dirs NUM_DIRS
                        Number of leaf-level directories to generate (approximate, default = 20).
  -e DIRS_DEPTH, --dirs-depth DIRS_DEPTH
                        Depth of leaf-level directories (default = 3).
  --dir-node-base DIR_NODE_BASE
                        Name base for node (non-leaf) directories (default = "node_").
  --dir-leaf-base DIR_LEAF_BASE
                        Name base for leaf (bottom level) directories (default = "leaf_").
  --file-base FILE_BASE
                        Name base for created files (default = "file_").
  --file-content FILE_CONTENT
                        Path to file with content to be inserted in created files (default empty).
  -n, --dry-run         Don't actually create any dirs/files, nor delete any existing tree.
  -v, --verbose         Print status and activity messages.
  -V, --version         Return version number and exit.
```

## Example run
```
$ ./genfiles . --num-files 20 --num-dirs 5 --dirs-depth 3 --verbose
Requested dirs_depth                     3
Requested num_dirs                       5
Requested num_files                      20
dir_node_base                            <node_>
dir_leaf_base                            <leaf_>
file_base                                <file_>
Calc num_bottom_node_dirs                4
Calc num_leafdirs_per_bottom_node_dir    2
Calc num_files_per_leafdir               4
Deleting existing tree <./gentree/>
./gentree/node_A/
./gentree/node_A/node_A/
./gentree/node_A/node_A/leaf_1/
./gentree/node_A/node_A/leaf_1/file_1
./gentree/node_A/node_A/leaf_1/file_2
./gentree/node_A/node_A/leaf_1/file_3
./gentree/node_A/node_A/leaf_1/file_4
./gentree/node_A/node_A/leaf_2/
./gentree/node_A/node_A/leaf_2/file_5
./gentree/node_A/node_A/leaf_2/file_6
./gentree/node_A/node_A/leaf_2/file_7
./gentree/node_A/node_A/leaf_2/file_8
./gentree/node_A/node_B/
./gentree/node_A/node_B/leaf_3/
./gentree/node_A/node_B/leaf_3/file_9
./gentree/node_A/node_B/leaf_3/file_10
./gentree/node_A/node_B/leaf_3/file_11
./gentree/node_A/node_B/leaf_3/file_12
./gentree/node_A/node_B/leaf_4/
./gentree/node_A/node_B/leaf_4/file_13
./gentree/node_A/node_B/leaf_4/file_14
./gentree/node_A/node_B/leaf_4/file_15
./gentree/node_A/node_B/leaf_4/file_16
./gentree/node_B/
./gentree/node_B/node_A/
./gentree/node_B/node_A/leaf_5/
./gentree/node_B/node_A/leaf_5/file_17
./gentree/node_B/node_A/leaf_5/file_18
./gentree/node_B/node_A/leaf_5/file_19
./gentree/node_B/node_A/leaf_5/file_20
Actual num bottom node dirs created      3
Actual num leaf dirs created             5
Actual num files created                 20
```
## Setup and Usage notes
- Supported on Python3 only.  Developed on Centos 7.8 with Python 3.6.8.
- The `RootPath` output directory is specified on the command line.  This directory must already exist.  The `gentree` subdirectory will be created at this location, and the directory and file hierarchy will be created beneath `gentree/`.
- _**On a successive run, everything in `<RootPath>/gentree/` is deleted and replaced with the new run output.**_ One use-model is to generate a tree, then generate another tree at a lower level of the first tree.
- A _node_ directory is an intermediate directory in the hierarchy, versus a _leaf_ directory which is a bottom level directory.  Files are created in the leaf directories only.
- A binary tree of directories is created, with leaf directories at the specified `dirs-depth`.  For example, `--dirs-depth 5` will produce a 4-level deep tree of node directories with the `num-dirs` number of leaf-level directories distributed beneath the bottom node dirs.
- Referring to the above example, `--dirs-depth 3` would request up to 4 _bottom node-level_ directories (2**(dirs-depth-1)), with the requested `--num-dirs 5` number of leaf-level dirs distributed here - 2, 2, and 1 - then truncating the forth bottom node-level directory since there are no more leaf dirs to be generated. Similarly, if the number of files divided by the actual number of generated leaf directories is not a whole number then there will be a partially populated directory and fewer than the requested number of directories may be produced.  In any case, the requested number of files are always produced. When the specified number of files have been created the remainder of the directory hierarchy is truncated.  

- For a fully balanced tree, set `num-dirs` to be a multiple of 2**(`dirs-depth`-1) and `num-files` to be a multiple of `num-dirs`.  For example, `--num-files 128 --num-dirs 64 --dirs-depth 5`.
- Setting `--num-dirs 1 --dirs-depth 1` creates all the files in a single directory.
- `--file-content` specifies the path to a file to copy when creating files.  If not specified, the output files contain zero bytes.
- Use the `--verbose` switch to output a listing of the created directory tree and files.
- Use the `--dry-run` switch to avoid thrashing your disk while experimenting with settings.
- The `--dir-node-base`, `--dir-leaf-base`, and `--file-base` switches allow setting the prefix names as desired.  Perhaps you want to experiment with long filenames and long directory paths.  


## Known issues:
- none

## Version history

- 200713 v0.0  New
