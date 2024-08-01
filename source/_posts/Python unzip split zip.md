---
title: Python unzip split zip
date: 2024-08-01
tags: 
    - Python
categories: Summary
copyright: true
---

> :pushpin:开发中遇到 `.0z1, .z02, ..., .zip`格式的分卷压缩包需要自动解压，尝试了`zipfile`和`py7zr`都不行，后来看了文档说暂时不持支持分卷解压，网络上多数的实现方案是建临时文件然后往里面追加写入，试了不太行，最后采用调命令行执行`7z`的命令。

<!--more-->

## :books: 直接上手

``` python
def unzip_split_zip(part_zip_list: list) -> Union[bool, Any]:
    """
    Unzip the split zip files
    :param part_zip_list: List of paths to the split zip files
    :return: unzip success or not and the unzip file path
    """

    # Find the main zip file
    main_zip_file = None
    for file in part_zip_list:
        if file.endswith('.zip'):
            main_zip_file = file
            break

    if main_zip_file is None:
        return False

    directory_path = os.path.dirname(main_zip_file)
    pre_name = main_zip_file.rsplit('.zip', 1)[0]
    command = ['7z', 'x', main_zip_file, f'-o{directory_path}']

    try:
        # TODO Python暂无直接支持分卷解压的包，需要本地下载7Zip且添加到系统Path，这里调用命令行执行分卷解压
        subprocess.run(command, check=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
        return True, pre_name + '.app'
    except subprocess.CalledProcessError as e:
        print(f"An error occurred: {e.stderr}")
        return False
    except Exception as e:
        print(f"An unexpected error occurred: {e}")
        return False
```
