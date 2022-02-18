# Competitive-programming-sereies
In this sereies i will give all the useful information regarding competitive programming.



 ______________________________________________*****************----------STL TIPS AND TRICKS-----------****************_________________________________________ 


Hello, everyone.

I would like to tell you about some tricks and constructions I use in C++. They are not hidden or anything, moreover, most of them are from stl or maybe well known among software engineers, but I often see codes where something is done by hand in, say, 5 lines, while in stl there is a function which does the same.

The things below are filtered by my personal sense of non-popularity (so no __builtin functions) and usage frequency (so there almost surely are similar things I don't know about) and are not really sorted in any reasonable order. Let's begin.



________________________________________*****************************************************_____________________________________________________________

________________________________________________all(x)___________________________________________________


This may be an exception to the rule of non-popularity -- this is quite widely used, but some next items will depend on all(x), so I define it here. So, I talk about

#define all(x) (x).begin(), (x).end()
Now sorting a vector looks like sort(all(vec)) instead of sort(vec.begin(), vec.end()).

However, it's not all about this define. Imagine you need to build a convex hull of a set of points. Usually the code looks approximately as follows:

vector<Point> getConvexHull(vector<Point> pts) {
    int min_pos = min_element(all(pts)) - pts.begin();
    swap(pts[0], pts[min_pos]);
    sort(pts.begin() + 1, pts.end(), &cmp);
    // ...
}
And it might not come to one's mind that pts.begin() + 1 is the same as 1 + pts.begin(), and hence the code can be rewritten as

vector<Point> getConvexHull(vector<Point> pts) {
    int min_pos = min_element(all(pts)) - pts.begin();
    swap(pts[0], pts[min_pos]);
    sort(1 + all(pts), &cmp);
    // ...
}
Be, however, careful with this, because this reduces code readability significantly. Thanks to my teammates AndreySergunin and WHITE2302 for showing me this 1 + all(...) trick.
 
___________________________________________________****************************************____________________________________________________________
 
-----------------------------------------------------------std::unique--------------------------------------------------
 
This is also well known, but not as well as I want, so I leave it here. std::unique(begin, end) goes from begin to end, removes consecutive duplicates and returns the end of the resulting iterator range. So, for example, if vec = {1, 1, 2, 2, 3, 2, 1}, then unique(all(vec)) makes it equal {1, 2, 3, 2, 1, ?, ?} (where ? is not known, but probably the last two symbols are 2 and 1) and returns vec.begin() + 5. Usually, to make a vector containing only its unique elements, one writes

 
________________________________________*****************************************************_____________________________________________________________
------------------------------------------------------sort(all(vec));
 
vec.resize(unique(all(vec)) - vec.begin());
or creates a function/define make_unique (which is, actually, also a name of some existing function in C++) which does so.

nxt()
I noticed that many Kotlin Heroes participants often have some functions in their templates like readLong(), readToken() and so on. I understand that this may be for the sake of similarity with C++ paradigm where you can just read stuff or maybe because val (n, k) = readLine()!!.split(' ').map(String::toInt) is too long and weird for writing it just to read two numbers. However, what doesn't often come to mind is that a similar function in C++ can simplify a lot of things (or maybe it's just a matter of taste). So the function may look as follows:

int nxt() {
    int x;
    cin >> x;
    return x;
}
If a usual reading a graph looks like this:

int n, m;
cin >> n >> m;
for (int i = 0; i < m; ++i) {
    int u, v;
    cin >> u >> v;
    --u, --v;
    g[u].push_back(v);
    g[v].push_back(u);
}
then with your little mriemd nxt() you can write the following:

int n = nxt(), m = nxt();
for (int i = 0; i < m; ++i) {
    int u = nxt() - 1, v = nxt() - 1;
    g[u].push_back(v);
    g[v].push_back(u);
}
However, be careful! If pts is a vector of points, then it's really better to read it in the following way:

for (int i = 0; i < (int)pts.size(); ++i) {
    cin >> pts[i].x >> pts[i].y;
}
or at least in the following C++17 way:

for (auto& [x, y] : pts) {
    cin >> x >> y;
}
But don't do the following:

for (int i = 0; i < (int)pts.size(); ++i) {
    pts[i] = {nxt(), nxt()};
}
because in the last implementation the order of nxt() calls is not defined.

Also you may change the type to long long or even make the function template, but I haven't really ever felt I need the last.

Big thanks to my former teammates savinov and sokian for introducing this to me in their team template back in 2015 when I joined them. nxt() will return later in this blog.

 
 
________________________________________*****************************************************_____________________________________________________________
 
--------------------------std::fill and std::iota
 
Many of you know that if one wants to fill a piece of memory with a specific byte, they can use memset. It's quite fast and can be used to fill some (usually 1-dimensional) C-arrays by zeroes and minus ones. What if you want to fill a container by ones? The answer is as easy as pie:

fill(all(vec), 1);
And if you need to fill it by consecutive numbers, you can use std::iota. Now the constructor of a Dsu class with two optimizations may look as follows:

int n;
vector<int> parent, rank;

Dsu(int _n): n(_n), parent(_n), rank(_n) {
    iota(all(parent), 0);
}
Here 0 denotes the value of *parent.begin(), and each next value is obtained from the previous one by the pre-increment.

________________________________________*****************************************************_____________________________________________________________
 
--------------------------------------std::generate
 
If you have a 0-ary function (that is, with no arguments) and want to fill a range by its calls, instead of writing a for loop you can call std::generate: filling a vector vec by random values (assume it's rand() calls) may look like

generate(all(vec), rand);
My favourite: to read n followed by n numbers you can write

int n = nxt();
vector<int> a(n);
generate(all(a), nxt);
or, if you don't need the value of n later, even

vector<int> a(nxt());
generate(all(a), nxt);
Bonus: the last three function have a _n analogs. Instead of the end iterator/pointer, fill_n, iota_n and generate_n take the size as the second argument. So if you want to read, say, first n numbers into a vector vec of size greater than n, you can use

generate_n(vec.begin(), n, nxt);
instead of a longer

generate(vec.begin(), vec.begin() + n, nxt);
Thanks to fdoer for telling this.

 
________________________________________*****************************************************_____________________________________________________________
 
------------------------------------std::rotate
 
Assume you write a profile dp. Or a 3-layer bfs. Or something else, where you need sometimes to cyclically shift a vector by k. If k=1 then one can write a loop of swaps:

for (int i = 0; i < (int)vec.size() - 1; ++i) {
    swap(vec[i], vec[i + 1]);
}
But if k>1 then the easiest way to do this manually is probably to create another vector, fill it accordingly and then set the initial vector to this. However a simple

rotate(vec.begin(), vec.begin() + k, vec.end());
can do the trick. This function works in such a way that after rotate(begin, middle, end) the element *middle becomes the first from begin to end.

 
________________________________________*****************************************************_____________________________________________________________
 
 
-----------------------------------std::merge
 
If you want to build a segment tree where each node contains a sorted list of values from the corresponding range then on its initialization you may need to merge two vectors several times (or if you implement your own merge sort). Some of you, probably, write some code like this:

for (int v = n - 1; v > 0; --v) {
    int i = 0, j = 0;
    while (i < (int)nodes[2 * v].size() || j < (int)nodes[2 * v + 1].size()) {
        if (j == (int)nodes[2 * v + 1].size() || (i < (int)nodes[2 * v].size() && nodes[2 * v][i] < nodes[2 * v + 1][j])) {
            nodes[v].push_back(nodes[2 * v][i++]);
        } else {
            nodes[v].push_back(nodes[2 * v + 1][j++]);
        }
    }
}
However, this code can be shortened:

for (int v = n - 1; v > 0; --v) {
    nodes[v].resize(nodes[2 * v].size() + nodes[2 * v + 1].size());
    merge(all(nodes[2 * v]), all(nodes[2 * v + 1]), nodes[v].begin());
}
Basically, std::merge takes 5 arguments: begin and end of one interval to merge, begin and end of the second interval to merge, and where to write the result. Don't forget to allocate the required memory (the purpose of .resize() in the code above)!

 
 
________________________________________*****************************************************_____________________________________________________________
 
-----------------------------------------Create a set from a vector
 
Not many of us know, but if you want to create a std::set from a vector and find out that set doesn't have a constructor of vector, you may go write a loop with an ok face, like:

set<int> S;
for (int x : a) {
    S.insert(x);
}
However, std::set has a constructor of two iterators, so one can write

set<int> S(all(a));
This is actually very natural and one can deduce that set should have such a constructor, but I, to my shame, didn't know it until WHITE2302 told me this recently.

Check if set or map have a key
This may be the most well known of above (as well as the least time-saving trick), but I still see that some experienced guys do the subj like

if (S.find(key) != S.end()) {
    // ...
}
However, set and map have a .count() method which returns 1 iff the key is in the container, otherwise 0:

if (S.count(key)) {
    // ...
}
The reason why it is called so and has an int type is that std::multiset and std::multimap also have this method, and for them the method returns the count of elements, which may exceed 1, obviously. Thanks to my former students dimaskovas and krock21 for enlightening me about this.

Of course, if you need to do something with the element, if it is present in the set (for example, erase it), it may be useful to actually call .find() method in order to erase by iterator, not by element (thus kind of caching one tree descent).

 
 
________________________________________*****************************************************_____________________________________________________________
 
--------------------------------------Check if an element occurs in a sorted sequence
 
Assume that you have, say, a sorted vector, and you want to check if it contains an element, but unlike the previous case, you don't care about the element itself. In this case you don't need to implement your own binary search like

bool has(const vector<int>& vec, int key) {
    int l = 0, r = vec.size();
    while (r > l + 1) {
        int m = (l + r) / 2;
        if (vec[m] <= key) {
            l = m;
        } else {
            r = m;
        }
    }
    return l + 1 == r && vec[l] == key; // not to fall on the empty vector
}
Instead, you can just use

if (binary_search(all(vec), key)) {
    // ...
}
I think the function doesn't need any explanation (it returns bool, so there is really not much else one can extract from this). Thanks to some guy who used this method in some easy problem in an SRM.

std::partition_point
If we talk about binary search: assume you have a vector and a predicate p(x) so that p(x) = true for all elements of some prefix of vector vec and false on all others. To find the first place where p(x) doesn't hold one can simply use

int pos = partition_point(all(vec), p) - vec.begin();
In other words, partition_point(begin, end, p) returns the first iterator iter that p(*iter) = false, assuming that the condition above holds, that is, p is true on some prefix of [begin, end). I didn't use this in competitive programming (yet), but once wanted to use it for my job. Thanks to fdoer again for sharing his wisdom.

!!
Assume you want to use a function which maps 0 to 0 and every non-zero number to 1 (for example, to count non-zero numbers on all prefix subrectangles). The most observant of you may have notices that this is simply a cast to bool. The easiest bool casting operator is !! and it works like this:

for (int i = 0; i < n; ++i) {
    for (int j = 0; j < m; ++j) {
        pref[i + 1][j + 1] = pref[i][j + 1] + pref[i + 1][j] - pref[i][j] + !!a[i][j];
    }
}
Afaik, a mnemonic rule to remember it (if needed) is "bang, bang, you're boolean".
         
         
________________________________________*****************************************************_____________________________________________________________
         

---------------------------------------Print a binary representation of a number
If you want to print, say, last 20 bits of a number, it's not necessary to write smth like

void show_binary(int n) {
    for (int i = 0; i < 20; ++i) {
        cout << !!(n & (1 << i));
    }
    cout << "\n";
}
Instead, one can simply write

cout << bitset<20>(n) << "\n";
However, you need to fix the number of bits to show (which is usually not a problem because in most of the cases you need it for debug purposes) and get used to the order where the lowest bit is the rightmost (which is actually a natural order for a binary representation of a number, but not so natural if you consider a mask as a sort of a boolean vector).

Also, if you want to print an octal or a hexadecimal representation of a number, you can simply write cout << oct << n << "\n" and cout << hex << n << "\n", respectively. To return back to normal decimal, use cout << dec.

lambda type deduction
Imagine you want to quickly write a lambda which calculates the number of ways to choose 2 elements from n. You may write something like this:

auto choose2 = [&](int n) {
    if (n <= 1) {
        return 0;
    } else {
        return 1ll * n * (n - 1) / 2;
    }
};
and see that the compiler cannot deduce the type of your lambda: in one place you return int and in another place you return long long. Ok, you may think, let's show the type explicitly. without auto:

function<long long(int)> choose2 = [&](int n) {
    if (n <= 1) {
        return 0;
    } else {
        return 1ll * n * (n - 1) / 2;
    }
};
To your surprise, the error doesn't disappear. You are not very happy, you replace 0 by 0ll and move forward feeling that you could surely have done this using some language tools.

Indeed, in the very first version you could actually explicitly specify the output type by a simple arrow:

auto choose2 = [&](int n) -> long long {
    if (n <= 1) {
        return 0;
    } else {
        return 1ll * n * (n - 1) / 2;
    }
};
Big thanks to my friends and colleagues Wild_Hamster, ArtDitel and fdoer again for showing me this construction (not only in this context, however).

         
         
________________________________________*****************************************************_____________________________________________________________
         
---------------------------------------------Introducing variables inside an if statement
Imagine the following: you have a function f(), which takes time to be computed, and if its value satisfies some condition, you want to somehow use it. You don't want to write

if (is_good(f())) {
    use_somehow(f());
}
since it costs two calls of f. You may write something like this:

int x = f();
if (is_good(x)) {
    use_somehow(x);
}
but this is not very clean and leaves a variable which is not used then, maybe under a potentially useful name. To avoid this, one can wrap all this into a block like this:

{
    int x = f();
    if (is_good(x)) {
        use_somehow(x);
    }
}
but a shorter version would do the following:

if (int x = f(); is_good(x)) {
    use_somehow(x);
}
This is possible since C++17.

The smallest element from three
I bet many, many of you at least once wrote something like

int x = min(min(a, b), c);
feeling that something you do in your life is not right (while you type this, you usually have time to think about such things). Things get worse if you want to get a minimum of four elements:

int x = min(min(min(a, b), c), d);
(the order or arguments and the way to regroup them does not matter). You may have written a variadic template in your, well, template, to call something like min(a, b, c, ...), but you may do this without any prewritten code. Just watch:

int x = min({a, b, c, d});
Credits to the Jinotega twitter for this.

I'm sure I forgot something. I also feel that this blog is like a quick start guide into stl. Still, I hope a lot of you will find something useful here. C++, as every other language (programming or natural), is evolving, after all, and we should keep our eyes on how we can use it efficiently.
         
         
         
         
         
         
