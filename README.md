# fixedwidth [![GoDoc](https://godoc.org/github.com/ianlopshire/go-fixedwidth?status.svg)](http://godoc.org/github.com/ianlopshire/go-fixedwidth) [![Report card](https://goreportcard.com/badge/github.com/ianlopshire/go-fixedwidth)](https://goreportcard.com/report/github.com/ianlopshire/go-fixedwidth) [![Go Cover](http://gocover.io/_badge/github.com/ianlopshire/go-fixedwidth)](http://gocover.io/github.com/ianlopshire/go-fixedwidth)

Package fixedwidth provides encoding and decoding for fixed-width formatted Data.

`go get github.com/ianlopshire/go-fixedwidth`

## Usage

### Struct Tags

The struct tag schema schema used by fixedwidth is: `fixed:"{startPos},{endPos},[{alignment},[{padChar}]]"`<sup id="a1">[1](#f1)</sup>.

The `startPos` and `endPos` arguments control the position within a line. `startPos` and `endPos` must both be positive integers greater than 0. Positions start at 1. The interval is inclusive. 

The `alignment` argument controls the alignment of the value within it's interval. The valid options are `default`<sup id="a2">[2](#f2)</sup>, `right`, and `left`. The `alignment` is optional and can be omitted.

The `padChar` argument controls the character that will be used to pad any empty characters in the interval after writing the value. The default padding character is a space. The `padChar` is optional and can be omitted.

Fields without tags are ignored.

### Encode
```go
// define some data to encode
people := []struct {
    ID        int     `fixed:"1,5"`
    FirstName string  `fixed:"6,15"`
    LastName  string  `fixed:"16,25"`
    Grade     float64 `fixed:"26,30"`
    Alive     bool    `fixed:"32,36"`
}{
    {1, "Ian", "Lopshire", 99.5, true},
}

data, err := fixedwidth.Marshal(people)
if err != nil {
    log.Fatal(err)
}
fmt.Printf("%s", data)
// Output:
// 1    Ian       Lopshire  99.50 true
```

### Decode
```go
// define the format
var people []struct {
    ID        int     `fixed:"1,5"`
    FirstName string  `fixed:"6,15"`
    LastName  string  `fixed:"16,25"`
    Grade     float64 `fixed:"26,30"`
    Alive     bool    `fixed:"31,36"`
    Github    bool    `fixed:"37,38"`
}

// define some fixed-with data to parse
data := []byte("" +
    "1    Ian       Lopshire  99.50 false f" + "\n" +
    "2    John      Doe       89.50 true  t" + "\n" +
    "3    Jane      Doe       79.50 false F" + "\n" +
    "4    Ann       Carraway  79.59 false T" + "\n")

err := Unmarshal(data, &people)
if err != nil {
    log.Fatal(err)
}

fmt.Printf("%+v\n", people[0])
fmt.Printf("%+v\n", people[1])
fmt.Printf("%+v\n", people[2])
fmt.Printf("%+v\n", people[3])
// Output:
//{ID:1 FirstName:Ian LastName:Lopshire Grade:99.5 Alive:false Github:false}
//{ID:2 FirstName:John LastName:Doe Grade:89.5 Alive:true Github:true}
//{ID:3 FirstName:Jane LastName:Doe Grade:79.5 Alive:false Github:false}
//{ID:4 FirstName:Ann LastName:Carraway Grade:79.59 Alive:false Github:true}
```

It is also possible to read data incrementally

```go
decoder := fixedwidth.NewDecoder(bytes.NewReader(data))
for {
    var element myStruct
    err := decoder.Decode(&element)
    if err == io.EOF {
        break
    }
    if err != nil {
        log.Fatal(err)
    }
    handle(element)
}
```

If you have an input where the indices are expressed in unicode codepoints, and
not raw bytes fixedwidth supports this. Your data must be UTF-8 encoded:

```go
decoder := fixedwidth.NewDecoder(strings.NewReader(data))
decoder.SetUseCodepointIndices(true)
// Decode as usual now
```

### Alignment Behavior

| Alignment | Encoding | Decoding |
| --------- | -------- | -------- |
| `default` | Field is left aligned | The padding character is trimmed from both right and left of value |
| `left` | Field is left aligned | The padding character is trimmed from right of value |
| `right` | Field is right aligned | The padding character is trimmed from left of value |

## Notes
1. <span id="f1">`{}` indicates an argument. `[]` indicates and optional segment [^](#a1)</span>
2. <span id="f2">The `default` alignment is similar to `left` but has slightly different behavior required to maintain backwards compatibility [^](#a2)</span> 

## Licence
MIT
