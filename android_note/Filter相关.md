# Filter

顺便看看，Android中过滤器是怎么实现的？
注释中提到，Filter一般通过继承Filterable实现

## 具体实现

这是SimpleAdapter出现的一个过滤首字母item的一个过滤器

``` java

    private class SimpleFilter extends Filter {

        @Override
        protected FilterResults performFiltering(CharSequence prefix) {
            FilterResults results = new FilterResults();

            if (mUnfilteredData == null) {
                mUnfilteredData = new ArrayList<Map<String, ?>>(mData);
            }

            if (prefix == null || prefix.length() == 0) {
                ArrayList<Map<String, ?>> list = mUnfilteredData;
                results.values = list;
                results.count = list.size();
            } else {
                String prefixString = prefix.toString().toLowerCase();

                ArrayList<Map<String, ?>> unfilteredValues = mUnfilteredData;
                int count = unfilteredValues.size();

                ArrayList<Map<String, ?>> newValues = new ArrayList<Map<String, ?>>(count);

                for (int i = 0; i < count; i++) {
                    Map<String, ?> h = unfilteredValues.get(i);
                    if (h != null) {

                        int len = mTo.length;

                        for (int j = 0; j < len; j++) {
                            String str = (String) h.get(mFrom[j]);

                            String[] words = str.split(" ");
                            int wordCount = words.length;

                            for (int k = 0; k < wordCount; k++) {
                                String word = words[k];

                                if (word.toLowerCase().startsWith(prefixString)) {
                                    newValues.add(h);
                                    break;
                                }
                            }
                        }
                    }
                }

                results.values = newValues;
                results.count = newValues.size();
            }

            return results;
        }

        @Override
        protected void publishResults(CharSequence constraint, FilterResults results) {
            //noinspection unchecked
            mData = (List<Map<String, ?>>) results.values;
            if (results.count > 0) {
                notifyDataSetChanged();
            } else {
                notifyDataSetInvalidated();
            }
        }
    }

```

通过`performFiltering`处理Result和`publishResults`返回数据进行刷新。


## Filter抽象类源码  

抽象类Filter中filter开启了对于数据的异步过滤操作。
寻找一下调用，发现AbsListView和AutoCompleteTextView 都有调用:
Abs中：  

``` java  

	// 判断类型
	if (mAdapter instanceof Filterable) {
                Filter f = ((Filterable) mAdapter).getFilter();
                // Filter should not be null when we reach this part
                if (f != null) {
                    f.filter(s, this);
                } else {
                    throw new IllegalStateException("You cannot call onTextChanged with a non "
                            + "filterable adapter");
                }
            }

```
Auto中:

``` java  

    protected void performFiltering(CharSequence text, int keyCode) {
        mFilter.filter(text, this);
    }

```

filter的调用通过一个限制词和接口，接口判断完成百分比。

filter:

``` java  

	public final void filter(CharSequence constraint, FilterListener listener) {
		// 锁
        synchronized (mLock) {
            if (mThreadHandler == null) {
            // 开一个线程进行 这个线程自带Looper
                HandlerThread thread = new HandlerThread(
                        THREAD_NAME, android.os.Process.THREAD_PRIORITY_BACKGROUND);
                thread.start();
                mThreadHandler = new RequestHandler(thread.getLooper());
            }

            final long delay = (mDelayer == null) ? 0 : mDelayer.getPostingDelay(constraint);
            // 回收池拿一个msg
            Message message = mThreadHandler.obtainMessage(FILTER_TOKEN);
    		// 包含了过滤的一切信息的对象
            RequestArguments args = new RequestArguments();
            // make sure we use an immutable copy of the constraint, so that
            // it doesn't change while the filter operation is in progress
          
            args.constraint = constraint != null ? constraint.toString() : null;
            args.listener = listener;
            message.obj = args;
    		
    		// 移除了之前未处理的消息
            mThreadHandler.removeMessages(FILTER_TOKEN);
            mThreadHandler.removeMessages(FINISH_TOKEN);
            mThreadHandler.sendMessageDelayed(message, delay);
        }
    }


```

处理request的handler：

``` java   

	 private class RequestHandler extends Handler {
        public RequestHandler(Looper looper) {
            super(looper);
        }
        
        /**
         * <p>Handles filtering requests by calling
         * {@link Filter#performFiltering} and then sending a message
         * with the results to the results handler.</p>
         *
         * @param msg the filtering request
         */
        public void handleMessage(Message msg) {
            int what = msg.what;
            Message message;
            switch (what) {
                case FILTER_TOKEN:
                    RequestArguments args = (RequestArguments) msg.obj;
                    try {
                    	// 处理过程
                        args.results = performFiltering(args.constraint);
                    } catch (Exception e) {
                        args.results = new FilterResults();
                        Log.w(LOG_TAG, "An exception occured during performFiltering()!", e);
                    } finally {
                    	// 返回值
                        message = mResultHandler.obtainMessage(what);
                        message.obj = args;
                        message.sendToTarget();
                    }

					// 结束调用
                    synchronized (mLock) {
                        if (mThreadHandler != null) {
                            Message finishMessage = mThreadHandler.obtainMessage(FINISH_TOKEN);
                            mThreadHandler.sendMessageDelayed(finishMessage, 3000);
                        }
                    }
                    break;
                case FINISH_TOKEN:
                    synchronized (mLock) {
                        if (mThreadHandler != null) {
                            mThreadHandler.getLooper().quit();
                            mThreadHandler = null;
                        }
                    }
                    break;
            }
        }
    }


```

处理结果的Handler：

``` java  

    private class ResultsHandler extends Handler {
        /**
         * <p>Messages received from the request handler are processed in the
         * UI thread. The processing involves calling
         * {@link Filter#publishResults(CharSequence,
         * android.widget.Filter.FilterResults)}
         * to post the results back in the UI and then notifying the listener,
         * if any.</p> 
         *
         * @param msg the filtering results
         */
        @Override
        public void handleMessage(Message msg) {
            RequestArguments args = (RequestArguments) msg.obj;
			// 返回数据
            publishResults(args.constraint, args.results);
            if (args.listener != null) {
                int count = args.results != null ? args.results.count : -1;
                args.listener.onFilterComplete(count);
            }
        }
    }


```

