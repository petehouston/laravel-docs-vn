# Eloquent: Collections

- [Giới thiệu](#introduction)
- [Danh sách các hàm](#available-methods)
- [Tạo collection riêng](#custom-collections)

<a name="introduction"></a>
## Giới thiệu

Tất cả các tập hợp nhiều kết quả bởi Eloquent đều là một instance từ `Illuminate\Database\Eloquent\Collection`, bao gồm kết quả lấy từ hàm `get` hay thông qua relationship. Các Eloquent collection object kế thừa từ Laravel [base collection](/docs/{{version}}/collections), nên chúng đề kế thừa các hàm để làm xử lý với lớp dưới của Eloquent model.

Tất nhiên là các collections đều có thể sử dụng như iterators, cho phép bạn thực hiện lặp như với một mảng PHP:

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

Tuy nhiên, collection còn mạnh hơn array nhiều và cung cấp thêm một số các xử lý map / reduce mà có thể móc nối được qua một interface dễ hiểu. Ví dụ, cùng nhau xoá các model inactive và thu thập tên của các user còn lại:

    $users = App\User::where('active', 1)->get();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> **Chú ý:** Trong khi hầu hết hàm Eloquent collection  trả về một instance mới của một Eloquent collection, các hàm `pluck`, `keys`, `zip`, `collapse`, `flatten` và `flip` trả về một instance của [base collection](/docs/{{version}}/collections).

<a name="available-methods"></a>
## Danh sách các hàm

### Base Collection

Tất cả các Eloquent collections đều kế thừa từ [Laravel collection](/docs/{{version}}/collections), vì thế mà chúng kế thừa tất cả các hàm mạnh mẽ được cung cấp bởi lớp base collection:

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">
[all](/docs/{{version}}/collections#method-all)
[chunk](/docs/{{version}}/collections#method-chunk)
[collapse](/docs/{{version}}/collections#method-collapse)
[contains](/docs/{{version}}/collections#method-contains)
[count](/docs/{{version}}/collections#method-count)
[diff](/docs/{{version}}/collections#method-diff)
[each](/docs/{{version}}/collections#method-each)
[every](/docs/{{version}}/collections#method-every)
[filter](/docs/{{version}}/collections#method-filter)
[first](/docs/{{version}}/collections#method-first)
[flatten](/docs/{{version}}/collections#method-flatten)
[flip](/docs/{{version}}/collections#method-flip)
[forget](/docs/{{version}}/collections#method-forget)
[forPage](/docs/{{version}}/collections#method-forpage)
[get](/docs/{{version}}/collections#method-get)
[groupBy](/docs/{{version}}/collections#method-groupby)
[has](/docs/{{version}}/collections#method-has)
[implode](/docs/{{version}}/collections#method-implode)
[intersect](/docs/{{version}}/collections#method-intersect)
[isEmpty](/docs/{{version}}/collections#method-isempty)
[keyBy](/docs/{{version}}/collections#method-keyby)
[keys](/docs/{{version}}/collections#method-keys)
[last](/docs/{{version}}/collections#method-last)
[map](/docs/{{version}}/collections#method-map)
[merge](/docs/{{version}}/collections#method-merge)
[pluck](/docs/{{version}}/collections#method-pluck)
[pop](/docs/{{version}}/collections#method-pop)
[prepend](/docs/{{version}}/collections#method-prepend)
[pull](/docs/{{version}}/collections#method-pull)
[push](/docs/{{version}}/collections#method-push)
[put](/docs/{{version}}/collections#method-put)
[random](/docs/{{version}}/collections#method-random)
[reduce](/docs/{{version}}/collections#method-reduce)
[reject](/docs/{{version}}/collections#method-reject)
[reverse](/docs/{{version}}/collections#method-reverse)
[search](/docs/{{version}}/collections#method-search)
[shift](/docs/{{version}}/collections#method-shift)
[shuffle](/docs/{{version}}/collections#method-shuffle)
[slice](/docs/{{version}}/collections#method-slice)
[sort](/docs/{{version}}/collections#method-sort)
[sortBy](/docs/{{version}}/collections#method-sortby)
[sortByDesc](/docs/{{version}}/collections#method-sortbydesc)
[splice](/docs/{{version}}/collections#method-splice)
[sum](/docs/{{version}}/collections#method-sum)
[take](/docs/{{version}}/collections#method-take)
[toArray](/docs/{{version}}/collections#method-toarray)
[toJson](/docs/{{version}}/collections#method-tojson)
[transform](/docs/{{version}}/collections#method-transform)
[unique](/docs/{{version}}/collections#method-unique)
[values](/docs/{{version}}/collections#method-values)
[where](/docs/{{version}}/collections#method-where)
[whereLoose](/docs/{{version}}/collections#method-whereloose)
[zip](/docs/{{version}}/collections#method-zip)
</div>

<a name="custom-collections"></a>
## Tạo collection riêng

Nếu bạn muốn tạo một object `Collection` riêng với hàm mở rộng riêng, bạn có thể ghi đè hàm `newCollection` trong model:

    <?php

    namespace App;

    use App\CustomCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Create a new Eloquent Collection instance.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

Khi bạn đã khai báo một hàm `newCollection`, bạn sẽ nhận được một instance của collection đó bất cứ khi nào Eloquent trả về một `Collection` của model. Nếu bạn muốn sử dụng collection riêng cho tất cả các model trong ứng dụng, bạn nên ghi đè vào hàm `newCollection` trong class base của model, mà được kế thừa từ tất cả các model khác.
