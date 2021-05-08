# go 位集合运算例子
来自 《Go 程序设计语言》 中 示例：位向量

## 代码
```go
// IntSet 是一个包含非负数整数的集合
// 零值代表空的集合
type IntSet struct {
	words []uint64
}

// Has方法的返回值代表是否存在非负数x
func (s *IntSet) Has(x int) bool {
	word, bit := x/64, uint(x%64)
	return word < len(s.words) && s.words[word]&(1<<bit) != 0
}

// Add 添加非负数 x 到集合中
func (s *IntSet) Add(x int) {
	word, bit := x/64, uint(x%64)
	for word >= len(s.words) {
		s.words = append(s.words, 0)
	}

	s.words[word] |= 1 << bit
}

// UnionWith 将会对 s 和 t 做并集将结果存在 s 中
func (s *IntSet) UnionWith(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] |= tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}

func (s *IntSet) Len() int {
	var totalLen int
	for _, word := range s.words {
		if word == 0 {
			continue
		}

		for j := 0; j < 64; j++ {
			if word&(1<<uint(j)) != 0 {
				totalLen++
			}
		}
	}

	return totalLen
}

// String 方法以字符串 "{1, 2, 3}" 的形式返回集中
func (s *IntSet) String() string {
	var buf bytes.Buffer
	buf.WriteByte('{')
	for i, word := range s.words {
		if word == 0 {
			continue
		}

		for j := 0; j < 64; j++ {
			if word&(1<<uint(j)) != 0 {
				if buf.Len() > len("{") {
					buf.WriteByte(' ')
				}

				fmt.Fprintf(&buf, "%d", 64*i+j)
			}
		}
	}

	buf.WriteByte('}')
	return buf.String()
}

func (s *IntSet) Remove(x int) {
	if !s.Has(x) {
		return
	}

	word, bit := x/64, x%64
	s.words[word] &= ^(1 << uint(bit))
}

func (s *IntSet) Clear() {
	for i := range s.words {
		s.words[i] &= 0
	}
}

func (s *IntSet) Copy() *IntSet {
	var dstIntSet IntSet
	dstIntSet.words = make([]uint64, len(s.words))
	copy(dstIntSet.words, s.words)

	return &dstIntSet
}
```