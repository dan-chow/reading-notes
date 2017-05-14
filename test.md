<table>
<tr><th colspan='2'>Ways to Initialize a vector</th></tr>
<tr><td>vector<T> v1</td><td>vector that holds objects of type T. Default initialization; v1 is empty.</td></tr>
<tr><td>vector<T> v2(v1)</td><td>v2 has a copy of each element in v1.</td></tr>
<tr><td>vector<T> v2 = v1</td><td>Equivalent to v2(v1), v2 is a copy of the elements in v1.</td></tr>
<tr><td>vector<T> v3(n, val)</td><td>v3 has n elements with value val.</td></tr>
<tr><td>vector<T> v4(n)</td><td>v4 has n copies of a value-initialized object.</td></tr>
<tr><td>vector<T> v5{a,b,c...}</td><td>v5 has as many elements as there are initializers; elements are initialized by corresponding initializers.</td></tr>
<tr><td width='170'>vector<T> v5 = {a,b,c...}</td><td>Equivalent to v5{a,b,c...}.</td></tr>
<table>