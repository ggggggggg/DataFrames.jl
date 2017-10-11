# Categorical Data

Often, we have to deal with factors that take on a small number of levels:

```jldoctest categorical
julia> v = ["Group A", "Group A", "Group A", "Group B", "Group B", "Group B"]
6-element Array{String,1}:
 "Group A"
 "Group A"
 "Group A"
 "Group B"
 "Group B"
 "Group B"

```

The naive encoding used in an `Array` represents every entry of this vector as a full string. In contrast, we can represent the data more efficiently by replacing the strings with indices into a small pool of levels. This is what the `CategoricalArray` type does:

```jldoctest categorical
julia> using CategoricalArrays

julia> cv = CategoricalArray(v)
6-element CategoricalArrays.CategoricalArray{String,1,UInt32}:
 "Group A"
 "Group A"
 "Group A"
 "Group B"
 "Group B"
 "Group B"

```

`CategoricalArrays` support missing values via the `Nulls` package.

```jldoctest categorical
julia> using Nulls

julia> cv = CategoricalArray(["Group A", null, "Group A",
                              "Group B", "Group B", null])
6-element CategoricalArrays.CategoricalArray{Union{Nulls.Null, String},1,UInt32}:
 "Group A"
 null
 "Group A"
 "Group B"
 "Group B"
 null
```

In addition to representing repeated data efficiently, the `CategoricalArray` type allows us to determine efficiently the allowed levels of the variable at any time using the `levels` function (note that levels may or may not be actually used in the data):

```jldoctest categorical
julia> levels(cv)
2-element Array{String,1}:
 "Group A"
 "Group B"

```

The `levels!` function also allows changing the order of appearance of the levels, which can be useful for display purposes or when working with ordered variables.

```jldoctest categorical
julia> levels!(cv, ["Group B", "Group A"]);

julia> levels(cv)
2-element Array{String,1}:
 "Group B"
 "Group A"

julia> sort(cv)
6-element CategoricalArrays.CategoricalArray{Union{Nulls.Null, String},1,UInt32}:
 "Group B"
 "Group B"
 "Group A"
 "Group A"
 null
 null

```

By default, a `CategoricalArray` is able to represent 2<sup>32</sup>differents levels. You can use less memory by calling the `compress` function:

```jldoctest categorical
julia> cv = compress(cv)
6-element CategoricalArrays.CategoricalArray{Union{Nulls.Null, String},1,UInt8}:
 "Group A"
 null
 "Group A"
 "Group B"
 "Group B"
 null

```

Often, you will have factors encoded inside a DataFrame with `Array` columns instead of `CategoricalArray` columns. You can do conversion of a single column using the `categorical` function:

```jldoctest categorical
julia> cv = categorical(v)
6-element CategoricalArrays.CategoricalArray{String,1,UInt32}:
 "Group A"
 "Group A"
 "Group A"
 "Group B"
 "Group B"
 "Group B"

```

Or you can edit the columns of a `DataFrame` in-place using the `categorical!` function:

```jldoctest categorical
julia> using DataFrames

julia> df = DataFrame(A = [1, 1, 1, 2, 2, 2],
                      B = ["X", "X", "X", "Y", "Y", "Y"])
6×2 DataFrames.DataFrame
│ Row │ A │ B │
├─────┼───┼───┤
│ 1   │ 1 │ X │
│ 2   │ 1 │ X │
│ 3   │ 1 │ X │
│ 4   │ 2 │ Y │
│ 5   │ 2 │ Y │
│ 6   │ 2 │ Y │

julia> eltypes(df)
2-element Array{Type,1}:
 Int64
 String

julia> categorical!(df, [:A, :B])
6×2 DataFrames.DataFrame
│ Row │ A │ B │
├─────┼───┼───┤
│ 1   │ 1 │ X │
│ 2   │ 1 │ X │
│ 3   │ 1 │ X │
│ 4   │ 2 │ Y │
│ 5   │ 2 │ Y │
│ 6   │ 2 │ Y │

julia> eltypes(df)
2-element Array{Type,1}:
 CategoricalArrays.CategoricalValue{Int64,UInt32}
 CategoricalArrays.CategoricalValue{String,UInt32}

```

Using categorical arrays is important for working with the [GLM package](https://github.com/JuliaStats/GLM.jl). When fitting regression models, `CategoricalArray` columns in the input are translated into 0/1 indicator columns in the `ModelMatrix` with one column for each of the levels of the `CategoricalArray`. This allows one to analyze categorical data efficiently.

See the [CategoricalArrays package](https://github.com/JuliaData/CategoricalArrays.jl) for more information regarding categorical arrays.
