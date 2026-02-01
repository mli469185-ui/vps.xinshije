openapi: 3.1.0
info:
  title: 自定义GPT文件及命令管理工具
  version: 1.0.0
  description: 一个多功能的自定义工具，允许GPT执行Shell命令、读写文件、管理目录、搜索文件内容等操作，以最大限度地帮助用户完成任务。
servers:
  # *** 重要提示：已经使用你提供的域名进行替换！ ***
  # 如果你的服务器后端实际监听在例如 8000 端口，且通过反向代理（如 Nginx）将 https://api.xiangkuntrade.com 转发到该端口，
  # 那么此处填写的域名就足够了。GPT将通过此地址与你的服务器通信。
  - url: https://api.xiangkuntrade.com
    description: 你的自定义工具服务器的基准URL。GPT将通过此地址与你的服务器通信。
paths:
  /command:
    post:
      summary: 执行Shell命令 (execute_command)
      description: 在服务器上执行一个给定的Shell命令，并返回其标准输出、标准错误和退出码。这是一个非常强大的功能，请谨慎使用，确保服务器端对命令有严格的验证和限制，以防安全风险。
      operationId: execute_command
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - cmd
              properties:
                cmd:
                  type: string
                  description: 要在服务器上执行的Shell命令字符串。
      responses:
        '200':
          description: 命令执行成功。
          content:
            application/json:
              schema:
                type: object
                properties:
                  stdout:
                    type: string
                    description: 命令的标准输出内容。
                  stderr:
                    type: string
                    description: 命令的标准错误输出内容。
                  returncode:
                    type: integer
                    description: 命令的退出码（通常0表示成功）。
        '500':
          description: 服务器内部错误，可能在执行命令时出错。
  
  /file/read:
    get:
      summary: 读取文件内容 (read_file)
      description: 读取服务器上指定路径文件的全部文本内容。可用于“读”文件。
      operationId: read_file
      parameters:
        - in: query
          name: path
          schema:
            type: string
          required: true
          description: 要读取的文件的完整路径（例如：/var/www/html/index.html）。
      responses:
        '200':
          description: 文件内容成功读取。
          content:
            application/json:
              schema:
                type: object
                properties:
                  content:
                    type: string
                    description: 文件的全部文本内容。
        '404':
          description: 指定的文件路径不存在。
        '500':
          description: 读取文件时服务器发生内部错误。
  
  /file/write:
    post:
      summary: 写入或追加文件内容 (write_file)
      description: 向服务器上指定路径的文件写入文本内容。如果文件不存在则创建，如果存在则根据`append`参数选择覆盖或追加。可用于“写”文件和“存取”数据。
      operationId: write_file
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - path
                - content
              properties:
                path:
                  type: string
                  description: 要写入的文件的完整路径。
                content:
                  type: string
                  description: 要写入或追加到文件的文本内容。
                append:
                  type: boolean
                  default: false
                  description: 如果设置为 `true`，内容将追加到文件末尾；如果为 `false` 或未提供，则覆盖文件现有内容。
      responses:
        '200':
          description: 文件内容成功写入或追加。
        '500':
          description: 写入文件时服务器发生内部错误。
  
  /file/delete:
    delete:
      summary: 删除文件 (delete_file)
      description: 删除服务器上指定路径的文件。可用于“存取”管理。
      operationId: delete_file
      parameters:
        - in: query
          name: path
          schema:
            type: string
          required: true
          description: 要删除的文件的完整路径。
      responses:
        '200':
          description: 文件成功删除。
        '404':
          description: 指定的文件路径不存在。
        '500':
          description: 删除文件时服务器发生内部错误。

  /directory/list:
    get:
      summary: 列出目录内容 (list_directory)
      description: 列出服务器上指定路径下目录的所有文件和子目录的名称。可用于“读”、“扫描”文件系统结构。
      operationId: list_directory
      parameters:
        - in: query
          name: path
          schema:
            type: string
          required: true
          description: 要列出内容的目录的完整路径（例如：/home/user/my_project）。
      responses:
        '200':
          description: 目录内容成功获取。
          content:
            application/json:
              schema:
                type: object
                properties:
                  files:
                    type: array
                    items:
                      type: string
                    description: 目录中所有文件和子目录的名称列表。
        '404':
          description: 指定的目录路径不存在。
        '500':
          description: 列出目录内容时服务器发生内部错误。

  /directory/create:
    post:
      summary: 创建目录 (create_directory)
      description: 在服务器上指定路径创建新目录。可用于“写”、“存取”管理。
      operationId: create_directory
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - path
              properties:
                path:
                  type: string
                  description: 要创建的目录的完整路径。
                recursive:
                  type: boolean
                  default: false
                  description: 如果为 `true`，则会创建路径中所有不存在的父级目录。
      responses:
        '200':
          description: 目录成功创建。
        '409':
          description: 目录已存在。
        '500':
          description: 创建目录时服务器发生内部错误。
  
  /file/search_content:
    post:
      summary: 在文件中搜索内容 (search_file_content)
      description: 在服务器上指定路径的文件中搜索匹配给定模式（支持正则表达式）的文本行。可用于“扫描”文件内容。
      operationId: search_file_content
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - path
                - pattern
              properties:
                path:
                  type: string
                  description: 要搜索的文件的完整路径。
                pattern:
                  type: string
                  description: 要搜索的正则表达式或字符串模式。
                case_sensitive:
                  type: boolean
                  default: false
                  description: 如果为 `true`，则进行区分大小写的搜索。
      responses:
        '200':
          description: 搜索完成。
          content:
            application/json:
              schema:
                type: object
                properties:
                  matches:
                    type: array
                    items:
                      type: string
                    description: 包含匹配模式的行列表。
                  line_numbers:
                    type: array
                    items:
                      type: integer
                    description: 匹配行对应的行号列表。
        '404':
          description: 指定的文件路径不存在。
        '500':
          description: 搜索文件内容时服务器发生内部错