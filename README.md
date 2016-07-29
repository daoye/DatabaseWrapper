# DatabaseWrapper
Simple database wrapper for Microsoft SQL Server and MySQL written in C#.  

For a sample app exercising this library, refer to the test project contained within the solution.

## Description
DatabaseWrapper is a simple database wrapper for Microsoft SQL Server nad MySQL databases written in C#.   

Core features:
- dynamic query building using expression objects
- support for nested queries within expressions
- support for SQL server native vs Windows authentication
- support for SELECT, INSERT, UPDATE, and DELETE, or raw queries
- built-in sanitization

## A Note on Sanitization
Use of parameterized queries vs building queries dynamically is a sensitive subject.  Proponents of parameterized queries have data on their side - that parameterization does the right thing to prevent SQL injection and other issues.  *I do not disagree with them*.  However, it is worth noting that with proper care, you CAN build systems that allow you to dynamically build queries, and you SHOULD do so as long as you build in the appropriate safeguards.

If you find an injection attack that will defeat the sanitization layer built into this project, please let me know!

## Simple Example
Refer to the test project for a more complete example.
```
using DatabaseWrapper;
DatabaseClient client = new DatabaseClient(DbTypes.MsSql, "localhost", 0, null, null, "SQLEXPRESS", "test");

// some variables we'll be using
Dictionary<string, object> d;
Expression e;
List<string> fields;
DataTable result;

// add a record
d = new Dictionary<string, object>();
d.Add("firstName", "Joel");
d.Add("lastName", "Christner");
d.Add("notes", "Author");
result = client.Insert("person", d);

// update a record
d = new Dictionary<string, object>();
d.Add("notes", "THE author :)");
e = new Expression { LeftTerm = "firstName", Operator = Operators.Equals, RightTerm = "Joel" };
result = client.Update("person", d, e);

// retrieve a record
fields = new List<string> { "firstName", "lastName" }; // leave null for *
e = new Expression { LeftTerm = "lastName", Operator = Operators.Equals, RightTerm = "Christner" };
result = client.Select("person", 0, fields, e, null);

// delete a record
e = new Expression { LeftTerm = "firstName", Operator = Operators.Equals, RightTerm = "Joel" };
result = client.Delete("person", e);
```

## Sample Compound Expression
Expressions can be nested in either the LeftTerm or RightTerm.  Conversion from Expression to a WHERE clause uses recursion, so you should have a good degree of flexibility in building your expressions in terms of depth.
```
Expression e = new Expression {
	LeftTerm = new Expression {
		LeftTerm = "age",
		Operator = Operators.GreaterThan,
		RightTerm = 30
	},
	Operator = Operators.And,
	RightTerm = new Expression {
		LeftTerm = "height",
		Operator = Operators.LessThan,
		RightTerm = 74
	}
};
```

## Other Notes
MySQL does not like to return updated rows.  Sorry about that.  I thought about making the UPDATE clause require that you supply the ID field and the ID value so that I could retrieve it after the fact, but that approach is just too limiting.