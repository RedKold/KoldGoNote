**表数据的增删改查**

- 添加数据
	- `put <table>, <rowkey>, <family:column>, <value>, <timestamp>`
- 查询数据
	- **查询某行数据**
	- `get <table>, <rowkey>, [<family:column>,...]`
	- 扫描表