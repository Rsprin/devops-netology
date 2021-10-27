# devops-netology

some changes

Файл terraform/.gitignore будет применен только к файлам которые находятся внутри директории terraform/

Будут проигнорированы файлы:

1. все файлы в директории .terraform/ которая находится в terraform/ и всех ее поддиректориях
2. все файлы с расширением .tfstate в terraform/ и всех ее поддиректориях
3. все файлы в имени которых есть .tfstate. в terraform/ и всех ее поддиректориях
4. файл crash.log в terraform/ и всех ее поддиректориях
5. все файлы с расширением .tfvars в terraform/ и всех ее поддиректориях
6. файлы override.tf и override.tf.json в terraform/ и всех ее поддиректориях
7. файлы которые заканчиваются на _override.tf и _override.tf.json в terraform/ и всех ее поддиректориях
8. файлы .terraformrc и terraform.rc  terraform/ и всех ее поддиректориях
