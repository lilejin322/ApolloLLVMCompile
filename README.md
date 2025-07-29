# Compile Baidu Apollo 6.0 | 7.0 | 8.0 | 9.0 using wllvm

This is a guide repository for compiling the [Baidu Apollo](https://github.com/ApolloAuto/apollo) source code using the [whole-program-llvm](https://github.com/travitch/whole-program-llvm) (wllvm) wrapper and automatically extracting LLVM bitcode (Intermediate Representation).

Currently, support has been implemented for four major versions of Apollo: 6.0, 7.0, 8.0, and 9.0.

I have tested it on four CPU-only machines, and no issues have occurred. If you encounter any issues, feel free to open an issue in this repository. I’ll respond as soon as I see it.

## Quickstart

1. Clone the specified branch of Baidu Apollo

    1. Apollo 6.0:
    ```bash
    git clone https://github.com/lilejin322/BaiduApollo.git -b r6.0.1_wllvm
    ```
    > What was changed? One-click diff: https://github.com/lilejin322/BaiduApollo/compare/r6.0.1%E2%80%A6r6.0.1_wllvm
    2. Apollo 7.0:
    ```bash
    git clone https://github.com/lilejin322/BaiduApollo.git -b r7.0.1_wllvm
    ```
    > What was changed? One-click diff: https://github.com/lilejin322/BaiduApollo/compare/r7.0.1%E2%80%A6r7.0.1_wllvm
    3. Apollo 8.0:
    ```bash
    git clone https://github.com/lilejin322/BaiduApollo.git -b r8.0.0_wllvm
    ```
    > What was changed? One-click diff: https://github.com/lilejin322/BaiduApollo/compare/r8.0.0%E2%80%A6r8.0.0_wllvm
    4. Apollo 9.0:
    ```bash
    git clone https://github.com/lilejin322/BaiduApollo.git -b r9.0.1_wllvm
    ```
    > What was changed? One-click diff: https://github.com/lilejin322/BaiduApollo/compare/r9.0.1%E2%80%A6r9.0.1_wllvm

> I’ve made minimal changes to the source code to avoid breaking its structure. Each major version has its own dedicated branch, which contains the known minimal patch set required.

2. Enter the project directory and pull the official Docker image for the corresponding Apollo version:
```bash
bash docker/scripts/dev_start.sh
```

3. Enter the Docker container using command:
```bash
bash docker/scripts/dev_into.sh
```

4. You’re now inside the Docker container. Since we’re switching to LLVM compilation, we need to install the missing libomp library inside the container (not on your host machine!):
```bash
sudo apt-get update
```
> I recommend separating the commands above and below. On Apollo 8.0, using && has been known to cause issues.
```bash
sudo apt-get install -y libomp-dev
```

5. Still inside the container, install wllvm using the default Python (usually Python 3.6 in Apollo images):
```bash
pip install wllvm
```

6. Inside the container, compile the project. I recommend using a CPU-only machine for now—I haven’t tested GPU builds, and since we’re only verifying C++ code, there’s no need for GPU compilation.
```bash
bash apollo.sh build_cpu
```

7. After successful compilation, you’ll find a bunch of Bitcode files with hashed filenames under `apollo/wllvm_bc`.

> These filenames look messy because wllvm stores them using hashes. To identify the original C++ source file, use llvm-dis to disassemble the .bc files into human-readable LLVM IR (.ll). Look for the source_file field in the IR to trace back to the original C++ source file.

## Technical Notes

1. Enabling Debug Mode in Bazel (demo in Apollo 7.0.1_wllvm)

I enabled Bazel’s debug compilation mode in the r7.0.1_wllvm branch. See commit:
https://github.com/lilejin322/BaiduApollo/commit/fcc2a6f0ac4ef8fba3418603cc8f6f237e15c0d0#diff-d98d938e279aaf8d3cc82a0ebc7a7b29fab693d033322b2f6a1c235793f0c676L59

This mode automatically appends the `-g` (debug info) and `-O0` (no optimization) flags during compilation.
You can switch to other optimization levels (like `-O1`) if needed. For other versions (6.0, 8.0, 9.0), you can enable debug mode by making similar source code edits.

2. Post-Compilation Analysis

Once compilation is complete, you can apply various static analysis frameworks on the generated Bitcode. Recommended tools include:

- **SVF**: https://github.com/SVF-tools/SVF

- **PhASAR**: https://github.com/secure-software-engineering/phasar

- **program-dependence-graph**: https://github.com/ARISTODE/program-dependence-graph

Feel free to explore others as well. Have fun experimenting!
