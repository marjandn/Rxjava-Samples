# Rxjava-Samples
there are some useful Rxjava operators(literally magical)that use in different condition

well i use [Kotlin](http://kotlinlang.org/) lang for these samples , but not so different with java version, there are some smaples
that i use untill now , so i very glad to with your recommendes,these samples will be more better and optimal :)


## TextWatcher
(for use this option you should use follow dependency)
``` 
implementation "com.jakewharton.rxbinding3:rxbinding:$rxBinding"
```
of JakeWharton [repo](https://github.com/JakeWharton/RxBinding)
```
RxTextView.afterTextChangeEvents(your_edittext_name)
                .filter { t -> t.getEditable().toString().length > 3 //some filter }
                .subscribe { //do sth if filter has pass }
                
RxTextView.beforeTextChangeEvents(edt_new_pass)
                .filter { t -> t.getEditable().toString().length > 3 }
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

also if you got "change ui in mainThread" error you should use debounce like bellow

```
   .debounce(1000, TimeUnit.MILLISECONDS, AndroidSchedulers.mainThread())
```

## FlatMap, SwitchMap and ConcatMap
i suggest you to read this [article](https://medium.com/appunite-edu-collection/rxjava-flatmap-switchmap-and-concatmap-differences-examples-6d1f3ff88ee0)
to fully understand the difference between these three operators


## for() for first element
for loop but break when first element pass filter block
we used this for loop type before
```
 for (i in 0 until offList.size) {
            if (offList[i].foodId == 2)
                doSomeFunc(offList[i])
        }
```
and now with [Rxjava operators](http://reactivex.io/documentation/operators) it be like bellow
```
Observable.fromIterable(offList)
                .filter{
                    it.foodId == 2
                }.firstElement()
                .subscribe{
                    doSomeFunc(it)
                }
```
or even can set for in for  
```
 Observable.fromIterable(mList)
                .filter{it ->
                    baseFoods.contains(it)
                }
                .map{it ->
                    Observable.fromIterable(baseFoods)
                            .filter{bf->
                                it.foodId == bf.foodId
                            }
                            .firstElement()
                            .subscribe{bf ->
                                doSomeFunc(bf)
                            }
                }.subscribe()
```

## CompositeDisposable 
with this amazing Rxjava class you can canceled your RxJava request Each stage of the run. 
for example i had an server request with Rxjava like bellow
```
 ServiceGenerator().getService().getSellerDetails()
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribeWith(object : DisposableObserver<ResponsePacket>() {

                    override fun onComplete() {
                        log("get data complete...!!")
                    }

                    override fun onNext(t: ResponsePacket) {
                        manageSellerDetailsResponse(t)
                        dialog.dismiss()
                    }

                    override fun onError(e: Throwable) {
                        log("getSellerDetails  $e")
                    }

                })
```
That worked very well but if the user stopped requesting it during the run, that mean call onDestroy() this activity that run Rxjava Request
app crashes !! with this ugly error 

" java.lang.IllegalStateException: Fatal Exception thrown on Scheduler."

so we can solve this with CompositeDisposable class like bellow
```
val compositeDisposable = CompositeDisposable()

  fun getSellerDetails(shop: Int) {
        dialog.show()

      compositeDisposable.add(
        ServiceGenerator().getService().getSellerDetails(session.getUID(), session.getToken(), shop)
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribeWith(object : DisposableObserver<ResponsePacket>() {

                    override fun onComplete() {
                        log("get data complete...!!")

                        //get all Product Categories
                        getSellerProdCat(shop)
                    }

                    override fun onNext(t: ResponsePacket) {
                        manageSellerDetailsResponse(t)
                        dialog.dismiss()
                    }

                    override fun onError(e: Throwable) {
                        log("getSellerDetails  $e")
                    }

                }))
     }
     
      override fun onDestroy() {
        viewMode.compositeDisposable.clear()
        super.onDestroy()
    }
```
## simple example for login chain conditions

```
compositeDisposable.add(Observable.just(edt_mobile.text.toString())
            .filter {
                if (checkMobile(it))
                    return@filter true
                else {
                    edt_mobile.error = getString(R.string.login01_error)
                    return@filter false
                }
            }.map {
                edt_password.text.toString()
            }.filter {
                if (it.isNotBlank() && it.length >= 4)
                    return@filter true
                else {
                    edt_password.error = getString(R.string.login02_error)
                    return@filter false
                }
            }.subscribe {
                goLogin(edt_mobile.text.toString(), edt_password.text.toString())
            }
        )
```

## 1 minute reverse timer (useful for otp pages)
```
   Disposable disposable1 = Observable.interval(1, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .take(61)
                .filter(aLong -> aLong > 0)
                .map(aLong -> 60 - aLong)
                .subscribe(aLong -> {
                    if (aLong < 60 && aLong > 0) {
                        String tim;
                        if (aLong < 10)
                            tim = "00:" + "0" + aLong.toString();
                        else
                            tim = "00:" + aLong.toString();
                            
                         Log.d("marjan",tim);
                    } else{
                        Log.d("marjan","marjan hoooooooray !!!!");
                    }
                });

        getCompositeDisposable().add(disposable1);
```

## 1 minute timer
```
        Disposable disposable1 = Observable.interval(1, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .take(60)
                .filter(aLong -> aLong > 0)
                .subscribe(aLong -> {
                   if (aLong <= 60) {
                        String tim;
                        if (aLong < 10)
                            tim = "00:" + "0" + aLong.toString();
                        else
                            tim = "00:"+aLong.toString();

                        getUtils().showLog(tim);
                    } else
                        getUtils().showLog("marjan hoooooooray !!!!");
                });
```

## Grouping the emissions by a specific key

```
val requestMatches = FootbalDataApiImp.getApi().getMatches()
            //returning matches one by one from api response list
            .flatMap { response -> Observable.fromIterable(response.matches) }
            //group matches by their dates
            .groupBy { match -> match.date }
            .flatMapSingle{
                //create a list with matches that have the same keys (in this example: same date)
                it.toList()
            }
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe{ matches ->
                //do something
            }
```

## Retry a failed network request after a few seconds
```
val disposable = NetworkService.getSomething()
            //retry the failed request after 15 seconds
            .retryWhen { it.delay(15, TimeUnit.SECONDS) }
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe{ data ->
                //do something
            }
``` 

## Zip operator

```
            //combine emission from two or more observables into a single observable with zip operator
            val usersDisposable  = Observable.zip(
                database.userDao().getUserById(user1_id),
                database.userDao().getUserById(user2_id),
                BiFunction { user1: User, user2: User ->
                    //do something with the given users
                    Pair(user1, user2) 
                }
            ).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe{ userPair ->
                    print(userPair.first)
                    print(userPair.second)
                    //do something
                }
```

## Buffer operator and FromArray operator
buffer():> Periodically gather items from an Observable into bundles and emit the bundles rather than emitting
items one at a time. notice that order is maintained in the buffer() operator.
fromArray():> Input: T[] / Output: Observable<T>.

```
           String[] strings = new String[]{"RxAndroid", "RxJava", "Git", "Android"};

                   Observable<String> stringObservable = Observable
                           .fromArray(strings)
                           .subscribeOn(Schedulers.io());

                   stringObservable
                           .buffer(2)
                           .observeOn(AndroidSchedulers.mainThread())
                           .subscribe(new Observer<List<String>>() {
                               @Override
                               public void onSubscribe(Disposable d) {}

                               @Override
                               public void onNext(List<String> strings) {
                                   Log.e(TAG, "onNext: results for bundle are:");
                                   for(String string: strings){
                                       Log.e(TAG, "onNext: " + string);
                                   }
                               }

                               @Override
                               public void onError(Throwable e) {}

                               @Override
                               public void onComplete() {}
                           });
```

## Distinct operator
distinct():> The Distinct operator filters an Observable by only allowing items through that have not already been emitted.

```
               Observable<User> userObservable = Observable
                           .fromIterable(DataSource.createUsersList())
                           .distinct(new Function<User, String>() {
                               @Override
                               public String apply(User user) throws Exception {
                                   return user.getName();
                               }
                           })
                           .subscribeOn(Schedulers.io())
                           .observeOn(AndroidSchedulers.mainThread());

                   userObservable.subscribe(new Observer<User>() {
                       @Override
                       public void onSubscribe(Disposable d) {}
                       @Override
                       public void onNext(User user) {
                           Log.e(TAG, "onNext: " + user.getName());
                       }
                       @Override
                       public void onError(Throwable e) {}
                       @Override
                       public void onComplete() {}
                   });