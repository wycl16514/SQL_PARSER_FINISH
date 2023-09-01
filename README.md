本节我们完成 SQL 解释器的最后一部分，它涉及到数据的删除和更改，首先我们看删除语句的解析。我们先看 delete 对应的语法：
```go
Delete -> DELETE FROM ID (where Predicate)?
```
从语法规则可以看出，delete 语句必须以关键字 DELETE , FROM 开始，然后接着的字符串必须要满足 ID 的定义，最后可能接着 where 关键字，然后进入 Predicate 的解析，我们看看代码实现,在 parser.go 中的 Delete 函数增加代码如下：
```go 
func (p *SQLParser) Delete() interface{} {
	/*
		第一个关键字 delete,第二个关键字必须 from
	*/
	p.checkWordTag(lexer.DELETE)
	p.checkWordTag(lexer.FROM)
	p.checkWordTag(lexer.ID)
	tblName := p.sqlLexer.Lexeme
	pred := query.NewPredicate()
	if p.isMatchTag(lexer.WHERE) {
		pred = p.Predicate()
	}
	return NewDeleteData(tblName, pred)
}
```
新增一个 delete_data.go 文件，添加代码如下：
```go
package parser

import "query"

type DeleteData struct {
	tblName string
	pred    *query.Predicate
}

func NewDeleteData(tblName string, pred *query.Predicate) *DeleteData {
	return &DeleteData{
		tblName: tblName,
		pred:    pred,
	}
}

func (d *DeleteData) TableName() string {
	return d.tblName
}

func (d *DeleteData) Pred() *query.Predicate {
	return d.pred
}
```
最后在 main.go 增加代码如下：
```go
func main() {
	sql := "DELETE FROM Customers WHERE CustomerName=\"Alfreds Futterkiste\""
	sqlParser := parser.NewSQLParser(sql)
	sqlParser.UpdateCmd()

}
```
以上代码的调试演示过程请在 B 站搜索 coding 迪斯尼查看相关视频。我们还剩下最后一个语句，那就是 update,先看看 update 语句对应的语法：
```go
Modify -> UPDATE ID SET Field EQUAL Expression (WHERE Predicate)?
```
首先它必须以关键字 update 开头，然后跟着的字符串必须满足 ID 的定义，然后跟着关键字 SET, 后面跟着的一系列字符串要满足 Field 的定义，其实这里 Field 对应列名，下面跟着等号关键字，等号后面则是一个计算表达式，在最后我们还得判断是否接着 where 关键字，如果有，我们还要解析 where 后面对应的表达式，我们看看对应代码实现：
```go
func (p *SQLParser) Modify() interface{} {
	p.checkWordTag(lexer.UPDATE)
	p.checkWordTag(lexer.ID)
	//获得表名
	tblName := p.sqlLexer.Lexeme
	p.checkWordTag(lexer.SET)
	_, fldName := p.Field()
	p.checkWordTag(lexer.ASSIGN_OPERATOR)
	newVal := p.Expression()
	pred := query.NewPredicate()
	if p.isMatchTag(lexer.WHERE) {
		pred = p.Predicate()
	}
	return NewModifyData(tblName, fldName, newVal, pred)
}
```
接下来增加一个文件 modify_data.go,添加如下代码：
```go 
package parser

import "query"

type ModifyData struct {
	tblName string
	fldName string
	newVal  *query.Expression
	pred    *query.Predicate
}

func NewModifyData(tblName string, fldName string, newVal *query.Expression, pred *query.Predicate) *ModifyData {
	return &ModifyData{
		tblName: tblName,
		fldName: fldName,
		newVal:  newVal,
		pred:    pred,
	}
}

func (m *ModifyData) TableName() string {
	return m.tblName
}

func (m *ModifyData) TargetField() string {
	return m.fldName
}

func (m *ModifyData) NewValue() *query.Expression {
	return m.newVal
}

func (m *ModifyData) Pred() *query.Predicate {
	return m.pred
}

```

到这里我们就基本完成了一个小型 SQL 解释器，更详细的调试演示和讲解请在 B 站参看 coding 迪斯尼
