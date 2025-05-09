# **PHP Webshell Detection - Opcode Processing**

This repository provides scripts designed to detect potential webshells in PHP source code by using opcode analysis. The detection process is divided into two primary steps:

**1. Converting PHP Source Code to Opcode**  

The `1_to-opcode.py` script processes PHP source code files to generate opcode representations. Opcode is a low-level abstraction of PHP code, offering a more granular perspective for identifying potential security risks, such as webshells.

**2. Processing and Analyzing the Opcode**  

The `2_process.py` script further processes the generated opcode files. This step involves extracting opcode instructions and their corresponding operands, referred to as Opcode Double-Tuples (ODTs). These ODTs are decoded and analyzed using filtering rules to identify suspicious patterns. By focusing on the opcode structure rather than the source code itself, this analysis provides valuable insights into the underlying behavior of the PHP code, aiding the detection of potential webshells.

Together, these steps establish a robust analytical method for detecting malicious PHP scripts by examining their opcode-level behavior.

## Prerequisites

Before running the scripts, ensure you have the following installed:

- **PHP 8.0**: The scripts require PHP 8.0 to be installed on your system.
- **VLD (Vulcan Logic Dumper) 0.18.0**: This PHP extension is used to generate opcode from PHP source code.

### Installing PHP 8.0 and VLD 0.18.0

1. **Install PHP 8.0**:
   
   - On Ubuntu/Debian:
     ```bash
     sudo apt update
     sudo apt install software-properties-common
     sudo add-apt-repository ppa:ondrej/php
     sudo apt update
     sudo apt install php8.0
     sudo apt install php8.0-dev php-pear build-essential
     ```
2. **Install VLD 0.18.0**:
   
   - Download and install the VLD extension from [PECL](https://pecl.php.net/package/vld) or compile it from source.
     
     ```bash
     curl -L -O https://pecl.php.net/get/vld-0.18.0.tgz
     tar -xzf vld-0.18.0.tgz
     cd vld-0.18.0
     phpize
     ./configure
     make
     sudo make install
     ```
   - Enable the extension in your `php.ini` file:
     
     ```bash
     sudo vi /etc/php/8.0/cli/php.ini
     ```
     
     Add the following line at the end of the file:
     
     ```ini
     extension=vld.so
     ```
     
     You can check if the VLD extension is installed correctly:
     
     ```bash
     php -m | grep vld
     ```
     
     If the VLD extension is not working properly, you may need to restart the service.

## Usage

### Step 1: Convert PHP Source Code to Opcode

Run the `1_to-opcode.py` script to convert PHP source code files to opcode.

1. Place your PHP source code files in the `./samples` directory.
2. Run the script:
   
   ```bash
   python3 1_to-opcode.py
   ```
3. The script will generate opcode files and save them in the ./opcode directory.
4. Any files that fail to process will be moved to the ./error directory.

### Step 2: Process the Opcode

Run the `2_process.py` script to process the opcode files generated in the previous step.

1. Ensure the opcode files are in the ./opcode directory.
2. Run the script:
   
   ```bash
   python3 2_process.py
   ```
3. The script will process the opcode files and save the results in the ./processed_opc directory.

### Directory Structure

- **./samples**: Contains the PHP source code files.
  
  - There are 26 sample files in `./samples`, with filenames starting with `source_` indicating benign PHP code. The other files are webshells.
- **./opcode**: Contains the opcode files generated by `1_to-opcode.py`.
- **./error**: Contains any PHP source code files that failed to convert to opcode.
- **./processed_opc**: Contains the processed opcode files generated by `2_process.py`.
- **./datasets**: Datasets that have been organized and publicly released by the author.

### Script Details

---

#### 1_to-opcode.py

```
•    Input: PHP source code files in `./samples`.

•    Output: Opcode files in `./opcode`.

•    Error Handling: Files that fail to convert are moved to `./error`.
```

•    The command to run is:

```bash
php -d vld.active=1 -d vld.execute=0 -d vld.format=1 -d vld.col_sep="|" -d vld.verbosity=0 -d vld.save_paths=0 -d vld.dump_paths=1 -d vld.skip_prepend=1 ./samples/sample.php
```

##### Explanation of Parameters

- **`-d vld.active=1`**: This enables the VLD extension to be active during execution. The `1` means that the extension is enabled and will capture the opcode during script execution.
- **`-d vld.execute=0`**: This disables the execution of PHP code during the analysis, meaning that only the opcode will be generated without actually executing the PHP script. This is useful for just analyzing the PHP code structure without running it.
- **`-d vld.format=1`**: This controls whether the output is formatted according to a custom format. When set to `1`, the output will follow the custom format where you can adjust the interval between columns. If set to `0` (default), it will use the standard format.
- **`-d vld.col_sep="|"`**: This defines the character to separate columns in the opcode output. The default is a tab character (`\t`), but you can change it to a custom character, such as the pipe (`|`), to make the output easier to parse.
- **`-d vld.verbosity=0`**: This sets the verbosity level of the output. The value `0` means minimal output, while higher values (1, 2, 3) provide more detailed information about the opcode.
- **`-d vld.save_paths=0`**: This controls whether the path information is saved to a file. When set to `0` (default), it will not save the paths. Setting this to `1` saves the paths to the output file.
- **`-d vld.dump_paths=1`**: This specifies whether to output code branches and paths. Setting this to `1` means that code branches and paths will be included in the opcode dump.
- **`-d vld.skip_prepend=1`**: This tells VLD to skip the files specified in the `auto_prepend_file` directive in the `php.ini` configuration. These files are prepended to the script before it runs. When set to `1`, VLD will not process these files in the opcode output. This option only works if `vld.execute=0` is also set.

##### Customization Options

Users can adjust these parameters according to their specific needs for opcode generation.

---

####  2_process.py

- **Input**: Opcode files in `./opcode`.
- **Output**: Processed opcode files in `./processed_opc`.

##### Processing Steps:

1) **Extracts relevant lines** from the opcode files.
2) Extract the **ODTs**. **Filters and decodes** the ODTs.
   - The filtering process includes three steps: opcode instruction filtering, operand filtering, and a combined judgment filtering ODTs.
   - For operand decoding, it includes two decoding methods: URL decoding and Base64 decoding. Decoding will be performed iteratively for three times to handle cases of nested encodings.
   - The rules are crafted based on expert knowledge, ensuring effective extraction of meaningful features while disregarding low-value opcodes.Importantly, these rules preserve the contextual semantic relationships within the code.
3) **Saves the processed results** to the output directory (`./processed_opc`).If you need to change the directory, please update the corresponding location in the script.
4) Users can customize the **filtering rules** according to their requirements. If you are not an expert, please use caution when modifying these rules.

---

#### Datasets

1) The `./datasets` directory contains zip archives organized by the author, which include both the opcode dataset and the processed results. These datasets are available for anyone to access and use.
2) The dataset consists of the latest samples (from December 2021 to December 2024) as well as older samples (from November 2021 and earlier). The archives distinguish these two categories using the `new` and `old` folders. Both parts can be combined for model training.
3) The dataset is of high quality, containing **5,001 webshell samples** meticulously collected by the author. These files are sourced from multiple channels, including public websites, cybersecurity forums, major cybersecurity competitions, and malicious scripts captured from real-world servers used by hackers. Many webshells use anti-detection techniques. The **5,936 benign PHP samples** are collected from open-source PHP frameworks such as WordPress and PHPCMS. The overall positive-to-negative sample ratio is approximately 1:1.2.
4) The complete dataset is publicly available through our [Zenodo repository](https://doi.org/10.5281/zenodo.14938302) or via the Releases section on the right. A small number of webshell examples are provided in the `./samples` folder for quick inspection.

**Important Notes:**
- 🔑 Source code archive password: `123`
- ⚠️ Antivirus software may flag the archives - add the extraction folder to exclusion list before unpacking
- The complete dataset should contain **5,001 webshell samples** and **5,936 benign samples**. If your count is lower, some files might have been quarantined by security software.
- 🛑 **Legal Compliance Notice**: This dataset is provided strictly for academic research purposes. Any misuse for illegal activities is expressly prohibited. By using this dataset, you agree to:
  - Use the materials solely for non-malicious, research-oriented purposes
  - Not attempt to exploit any vulnerabilities contained in the samples
  - Assume full responsibility for any consequences resulting from misuse
  - Comply with all applicable laws and regulations in your jurisdiction

### Cite

Curated by SSL Security Lab, this comprehensive dataset embodies our decade-long dedication to advancing cybersecurity research. As a leading research institution in network and system security, SSL Security Lab remains at the forefront of academic exploration and technological innovation, consistently delivering cutting-edge advancements through peer-reviewed publications and practical security solutions.

Our team specializes in emerging security paradigms. This dataset has been meticulously compiled through real-world security assessments and experimental validations, adhering to rigorous academic standards while maintaining practical relevance to contemporary cybersecurity challenges.

We cordially invite researchers, institutions, and industry partners to leverage this resource for collaborative research initiatives. SSL Security Lab welcomes opportunities to co-develop next-generation security frameworks and address the evolving threats in our increasingly connected digital ecosystem. Let's collaborate to shape the future of cybersecurity.

This work has been accepted as a poster at the **[Network and Distributed System Security (NDSS) Symposium 2025 Accepted Posters](https://www.ndss-symposium.org/ndss2025/accepted-posters/)**, with the full preprint available on arXiv. We welcome citations to our paper or dataset in your research:

```bibtex
@misc{ndss2025-poster-46,
    author = {Zhiqiang Wang and Haoyu Wang and Lu Hao},
    title = {Poster: Long PHP webshell files detection based on sliding window attention},
    year = {2025},
    archivePrefix = {arXiv},
    eprint = {2502.19257},
    note = {Presented at NDSS 2025, poster available: \url{https://www.ndss-symposium.org/wp-content/uploads/2025-poster-46.pdf}},
    url = {https://doi.org/10.48550/arXiv.2502.19257}
}
```

### Reference

Part of the webshell dataset comes from public websites. Some URLs are listed below:

| No. | URL                                                                 |
|-----|---------------------------------------------------------------------|
| 1   | https://github.com/Cyc1e183/PHP-Webshell-Dataset                   |
| 2   | https://github.com/bartblaze/php-backdoors                         |
| 3   | https://github.com/tennc/Webshell                                  |
| 4   | https://github.com/cpkarma/Reborn-PHP-Bypass-Webshell              |
| 5   | https://github.com/Alexspedo168/Keqing-Bypass-Shell                |
| 6   | https://github.com/mIcHyAmRaNe/wso-webshell                        |
| 7   | https://github.com/22XploiterCrew-Team/Gel4y-Mini-Shell-Backdoor   |
| 8   | https://github.com/awsox/403-bypass-webshells                      |
| 9   | https://github.com/script044/webshells                             |
| 10  | https://github.com/cli-ish/non-alphanumeric-webshell               |
| 11  | https://github.com/Macr0phag3/webshell-bypassed-human              |
| 12  | https://github.com/rebeyond/Behinder                               |
| 13  | https://github.com/AntSwordProject/antSword                        |
| 14  | https://github.com/BeichenDream/Godzilla                           |

### License

This work is licensed under a Creative Commons Attribution-NonCommercial 4.0 International License.
To view details of this license, visit [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).
