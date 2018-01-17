# 批量复制并过滤指定文件
  shell 脚本如下
```
#!/bin/bash
#filename:cp_file
#eg:sh cp_file.sh '/d/projects/node/source/' '/d/projects/node/product/' '-path './task/excludefile' -o -path './task/exclu.log' -o -path './task/testfile/testfile1/testfile3''

#source_dir='/d/projects/node/source/'
source_dir=$1
#product_dir='/d/projects/node/product/'
product_dir=$2

#exclude_file=".excludefile"
# 编辑过滤文件
#exclude_file_name='-path './task/excludefile' -o -path './task/exclu.log' -o -path './task/testfile/testfile1/testfile3''
exclude_file_name=$3

echo "source dir: $source_dir"
echo "product dir: $product_dir"

#if [ ! -f "$exclude_file" ];then
# echo "请先创建过滤文件$exclude_file"
# exit 1
#fi

if [ ! -d "$product_dir" ];then
 mkdir -p "$product_dir"
fi

cd $source_dir


find ./ \( $exclude_file_name \)  -prune -o -name '*.*' -print > $product_dir.outcpfile.log

find ./ \( $exclude_file_name \)  -prune -o -type d -print  > $product_dir.outmkdir.log

#进入新目录

if [ -f "$product_dir.outmkdir.log" ];then

   for i in $(cat "$product_dir.outmkdir.log")
   do
       if [ ! -d "$product_dir$i" ];then
          echo "mkdir  $product_dir$i"
          mkdir "$product_dir$i" -p
       fi
   done

else
   echo ""$product_dir.outmkdir.log" 文件不存在"
   exit 1
fi

if [ -f "$product_dir.outcpfile.log" ];then

   for i in $(cat "$product_dir.outcpfile.log")
   do
       if [ ! -d "$product_dir$i" ];then
          cp $i $product_dir$i -f
          echo "cp $i to  $product_dir$i"
       fi
   done

else
   echo ""$product_dir.outcpfile.log" 文件不存在"
   exit 1
fi
```
