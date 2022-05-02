# `regexp`









```go
func MatchString(pattern string, s string) (matched bool, err error)
```

`MatchString()` сообщает, содержит ли *string* `s` какое-либо соответствие *regular expression* `pattern`. Более сложные запросы должны использовать `Compile()` и полный `Regexp` *interface*.

```go
func main() {
	matched, err := regexp.MatchString(`foo.*`, "seafood")
	fmt.Println(matched, err)  // true <nil>
	matched, err = regexp.MatchString(`bar.*`, "seafood")
	fmt.Println(matched, err)  // false <nil>
	matched, err = regexp.MatchString(`a(b`, "seafood")
	fmt.Println(matched, err)  // false error parsing regexp: missing closing ): `a(b`
}
```

