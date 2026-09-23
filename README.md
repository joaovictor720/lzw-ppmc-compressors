# LZW and PPMc Compressors

C++ implementations of lossless data compression algorithms developed for the **Introduction to Information Theory** course (*Introdução à Teoria da Informação*) at the **Federal University of Paraíba (UFPB)**.

The project explores two approaches to compression: **LZW (Lempel–Ziv–Welch)** builds a dictionary of repeated byte sequences, while **PPMc (Prediction by Partial Matching, method C)** uses preceding symbols as context for a statistical model and encodes symbols with arithmetic coding. Both implementations support compression, decompression, and saving or loading trained models.

## Build

You need a C++ compiler (`g++`) and Bash. From the repository root:

```bash
bash build.sh
```

This builds `jvav_lzw` from `source/LZW2/` and `jvav_ppmc` from `source/PPMc/source/`. The earlier LZW implementation in `source/LZW/` is also preserved.

If your compiler reports missing `uint64_t` or `uint8_t` declarations, build with an explicit include for `<cstdint>`:

```bash
g++ -include cstdint source/LZW2/*.cpp -o jvav_lzw -Ofast
g++ -include cstdint source/PPMc/source/*.cpp -o jvav_ppmc -Ofast
```

## Quick demo

Create a small input file:

```bash
printf 'TOBEORNOTTOBEORTOBEORNOT\n' > sample.txt
```

Compress and restore it with LZW:

```bash
./jvav_lzw -c sample.txt sample.lzw
./jvav_lzw -d sample.lzw restored-lzw.txt
cmp sample.txt restored-lzw.txt
```

Try PPMc with a maximum context length of 3:

```bash
./jvav_ppmc sample.txt -c 3
mv sample.txt.jvav3 restored-ppmc.txt.jvav3
./jvav_ppmc restored-ppmc.txt.jvav3 -d 3
cmp sample.txt restored-ppmc.txt
```

`cmp` produces no output when the restored file matches the original. PPMc appends `.jvav<context>` when compressing and removes that suffix when decompressing; renaming the compressed file in this example preserves the original input.

## Experiments

- **LZW:** `-m <limit>` limits dictionary entries, `-r` resets the dictionary when full, and `-b` records data for compression plots. Use `-sm <file>` to save a model or `-um <file>` to load one.
- **PPMc:** the positional context length controls the model order; `-m <limit>` limits the number of input bytes used to update the model. `-sm` saves it as `<input>.model`, and `-um <file>` loads a model.

Use matching model settings for compression and decompression, including the context length for PPMc and any dictionary limits or reset options for LZW.

The `silesia/` directory contains corpus files for larger experiments. The LZW helper script reports compressed size and average encoded bits per original byte:

```bash
bash get_entropy.sh --test sample.txt --lzw-max 4096
```

This helper requires `bc`. Add `-b demo` to generate a plot using `plotter.py`, which requires `python`, NumPy, and Matplotlib. Despite the script's name, the reported value is an empirical compression rate, not a direct calculation of Shannon entropy.
