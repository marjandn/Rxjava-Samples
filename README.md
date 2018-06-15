# Rxjava-Samples
there are some useful Rxjava operators(literally magical)that use in different condition

well i use [Kotlin](http://kotlinlang.org/) lang for these samples , but not so different with java version, there are some smaples
that i use untill now , so i very glad to with your recommendes,these samples will be more better and optimal :)


## TextWatcher
we can use 
```
RxTextView.afterTextChangeEvents(your_edittext_name)
                .filter { t -> t.toString().length > 3 //some filter }
                .subscribe { //do sth if filter has pass }
                
RxTextView.beforeTextChangeEvents(edt_new_pass)
                .filter { t -> t.toString().length > 3 }
                .subscribe { checkLayoutAccept() }
```
instead
```
et1.addTextChangedListener(new TextWatcher() {...}
```
## Do Sth base of each view
sometime i had to do some fuctions based on several views,
for example based on some CheckBoxes that are Checked or UnChecked and represents Days of week ,
saved a string contains days that Checked like "0123456" 
```
 var days = ""
        Observable.just(activity.check_sat_day, activity.check_sun_day, activity.check_mon_day, activity.check_tue_day, activity.check_wed_day, activity.check_thu_day, activity.check_fri_day)
                .filter({
                    it.isChecked
                }).flatMap {
                    if (it == activity.check_sat_day)
                        days += "0"
                    if (it == activity.check_sun_day)
                        days += "1"
                    if (it == activity.check_mon_day)
                        days += "2"
                    if (it == activity.check_tue_day)
                        days += "3"
                    if (it == activity.check_wed_day)
                        days += "4"
                    if (it == activity.check_thu_day)
                        days += "5"
                    if (it == activity.check_fri_day)
                        days += "6"

                    return@flatMap Observable.just(days)
                }.subscribe()
```
## Do Sth base of each Char in a String
This section is part of the previous section , that change isChecked property of those CheckBoxes based on stored string
so should seprate each char and set isCheck property of each CheckBox base of that (true or false)
```
 val ary: List<String> = days.split("".toRegex())//"12345" -> "1" , "2" , "3"
        Observable.fromIterable(ary)
                .flatMap {
                    if (it == "0")
                        check_sat_day.isChecked = true
                    if (it == "1")
                        check_sun_day.isChecked = true
                    if (it == "2")
                        check_mon_day.isChecked = true
                    if (it == "3")
                        check_tue_day.isChecked = true
                    if (it == "4")
                        check_wed_day.isChecked = true
                    if (it == "5")
                        check_thu_day.isChecked = true
                    if (it == "6")
                        check_fri_day.isChecked = true

                    return@flatMap Observable.just(it)
                }.subscribe()
```
## debounce on input field
sometimes i had to do some fuctions when user enter some string on a EditText BUT not on each char that user input
for example when you want to set user username field, each username that user input should go to server and check in db
to not be duplicate, so its not wisely to for each char that user input send a request for check input string
so we use debounce operator
```
Observable<String> obs;

        obs = RxTextView.textChanges(activity.getBinding().edtUsername).
                filter(charSequence ->
                {
                        Log.e("filter run  ", charSequence.toString());
                        return charSequence.length() > 3 && !baseUsername.equals(charSequence);
                })
                .debounce(2000, TimeUnit.MILLISECONDS)
                .map(charSequence -> {
                        Log.e("map run  ", charSequence.toString());
                        return charSequence.toString();
                });
        obs.subscribe(s -> {
                Log.e("obs.subscribe run  ", s);
                if (!s.equals(baseUsername))
                    checkUsername(s);
        });
```
and in below sample i saved and deleted based on entered string
```
 RxTextView.afterTextChangeEvents(holder!!.binding.edtOff)
                .debounce(1000, TimeUnit.MILLISECONDS)
                .observeOn(rx.android.schedulers.AndroidSchedulers.mainThread())
                .map {
                    val text = it.view().text.toString()
                    if (!text.isBlank()) {
                        if (text.trim().toInt() > 0) {
                            //do some function when editText has some values
                        }
                    } else {
                        //when user has cleared edtText values
                    }
                    return@map rx.Observable.just(text)
                }.switchMap {
                    return@switchMap it
                }
                .subscribe()

```

## FlatMap, SwitchMap and ConcatMap
i suggest you to read this [article](https://medium.com/appunite-edu-collection/rxjava-flatmap-switchmap-and-concatmap-differences-examples-6d1f3ff88ee0)
to fully understand the difference between these three operators



