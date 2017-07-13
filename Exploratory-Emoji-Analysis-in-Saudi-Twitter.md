
<div style="direction:rtl; font-size:30px; text-align:center; font-weight: bold;" > 
تحليل استكشافي لاستخدام الوجوه التعبيرية في المجتمع السعودي (عبر موقع تويتر)
</div>

<div style="direction:rtl; font-size:20px; text-align:center; font-style: italic" > 

بقلم: فهد الحازمي (<a href="https://twitter.com/fahd09">Twitter</a> | <a href="https://github.com/fahd09">Github</a>)

</div>

<div style="direction:rtl"> 
شكل الانتشار العالمي للوجوه التعبيرية - التي تعرف بالايموجي- بداية ثورة جديدة في فضاء اللغة الالكترونية، حيث أصبحت وسيلة لغوية وتواصلية لا غنى عنها خصوصاً لدى الأجيال الشابة.

في هذا التحليل نحاول أن نلتمس أبعاد هذ التغير في أحد أشهر مواقع التواصل الالكتروني، تويتر. حيث سنقوم بقراءة عدد ضخم من التغريدات ومن كافة مناطق المملكة العربية السعودية ومن ثم البحث عن الوجوه التعبيرية وتحليل ما يمكن أن تعكسه من متغيرات اجتماعية.

</div>

<div style="direction:rtl"> 
سنقوم أولاً باستيراد المكتبات اللازمة في بايثون وتثبيت بعض المتغيرات..
</div>


```python
import re
import numpy as np
import pandas as pd

# Visualization
import matplotlib.pyplot as plt
import seaborn as sns

plt.style.use('seaborn')
%matplotlib inline


from sklearn.feature_extraction.text import CountVectorizer
import nltk
```

<div style="direction:rtl"> 
تم جمع التغريدات كالتالي. أولاً، جهزت قائمة بأسماء بعض المدن مع خطوط الطول وخطوط العرض. ثم تم استخدام الواجهة البرمجية لموقع تويتر (Twitter Search API) مدعوماً بمكتبة (Tweepy) في بايثون لقراءة آلاف التغريدات التي تحتوي على أي وجه تعبيري ضمن قائمة 74 وجه تعبيري شائع. ثم حفظت نتائج البحث في جدول مستقل يحتوي على المدينة، تاريخ التغريدة، نص التغريدة وأخيراً مصدر التغريدة (البرنامج المستخدم لكتابتها). بالطبع، لا غنى عن مكتبة pandas في التعامل مع الجداول.
<br /> <br />
يلي استيراد قاعدة البيانات طباعة رأس الجدول لنتأكد من سلامة البيانات..
</div>


```python
tweets = pd.read_csv('./data/tweets_emojis_dataset_74emojis.csv', encoding='utf-8')
```


```python
tweets.shape
```




    (140656, 4)




```python
tweets.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>created_at</th>
      <th>text</th>
      <th>source</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Riyadh</td>
      <td>Wed Jul 12 18:47:24 +0000 2017</td>
      <td>#امسيه_منيف_الخمشي\n\nيكفي انها كلمات مؤثرة تت...</td>
      <td>Twitter Web Client</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Riyadh</td>
      <td>Wed Jul 12 18:46:49 +0000 2017</td>
      <td>وإن أحبُّوك ألفاً فلن يحبُّوك إلا قِطرة من بحري❤️</td>
      <td>Twitter for iPhone</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Riyadh</td>
      <td>Wed Jul 12 18:46:49 +0000 2017</td>
      <td>البيت الاخير يختصر كل المعاناة ❤️❤️😢 https://t...</td>
      <td>Twitter for iPhone</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Riyadh</td>
      <td>Wed Jul 12 18:46:31 +0000 2017</td>
      <td>هلا هلا ❤❤😢 miss you \nالطقم حكايه ثانيه افضل ...</td>
      <td>Twitter for Android</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Riyadh</td>
      <td>Wed Jul 12 18:46:30 +0000 2017</td>
      <td>قلب قلببببي❤️❤️❤️❤️.</td>
      <td>Twitter for iPhone</td>
    </tr>
  </tbody>
</table>
</div>



<div style="direction:rtl"> 
كلنا نعرف كمية التغريدات المزعجة (السبام) في تويتر العربي. ابتدا من " تبي ينلحس مخك؟" إلى الإعلانات الغثيثة ومروراً بحسابات الأدعية والأذكاء والتي سرعان ما تصل إلى أعداد متابعين كبيرة لتُباع مرة أخرى لمن أحب بريق الشهرة. على أية حال، هنا طريقة بسيطة وفعالة لفلترة التغريدات المزعجة وذلك بتحديد مصدر التغريدة. إن كنت مهتماً بمجال تحليل تغريدات تويتر وتواجه مشكلة في التغريدات المزعجة فلدي أفكار أخرى قليلة لا أمانع من مشاركتها، ولكن هذه الطريقة أدنا (الفلترة بالمصدر) هي الطريقة الأكثر فعالية.
</div>


```python
good_sources = ['Twitter Web Client', 'Twitter for iPhone', 'Twitter for Android', 'Twitter for iPad', 'Twitter Lite']
```


```python
tweets = tweets[tweets.source.isin(good_sources)]
```

<div style="direction:rtl"> 
كإجراء احتياطي، سنقوم بحذف الصفوف المكررة إن كان هناك صفوف مكررة حتى يحوي كل صف على تغريدة فريدة وغير مكررة. يلي ذلك طباعة عدد التغريدات في الجدول ألا وهو 134763 وهو عدد لا بأس به لغرض الاستكشاف
</div>


```python
tweets.drop_duplicates(inplace=True)
tweets.reset_index(inplace=True, drop=['index'])
```


```python
print('Total number of tweets: ', len(tweets))
```

    Total number of tweets:  134763


<div style="direction:rtl; font-size:25px; text-align:right; font-weight: bold;" > 
تنظيف البيانات
</div>

<div style="direction:rtl"> 
في الخانات التالية، سنقوم بأصعب مهمة في مجال تحليل البيانات، ألا هي تنظيف البيانات. ما يزيد المهمة صعوبة هو تنظيف النصوص العربية نظراً لوجود الكثير من الأخطاء الإملائية في النصوص العربية والزخارف وما إلى ذلك.

<br /> <br /> 

هناك الكثير من الخطوات التي يمكن عملها في مهمة تنظيف نص التغريدات ولكني اقتصرت على خطوات أساسية أعتقد أنها كافية لإعطاء نتائج على قدر لا بأس به من الدقة للمجالات العامة. بالمناسبة، نحن لن نحتاج إلى أي نصوص في هذه التدوينة سوى الوجوه التعبيرية ( والتي لا تستدعي كل هذه الخطوات على أية حال). ولكن لأني أيضاً قمت بالعديد من التحليلات الجانبية المتعلقة بنصوص التغريدات والتي قد لا يتسع المجال للحديث عنها.

</div>


```python
def denoise_arabic(text):
    noise = re.compile(""" ّ    | # Tashdid
                             َ    | # Fatha
                             ً    | # Tanwin Fath
                             ُ    | # Damma
                             ٌ    | # Tanwin Damm
                             ِ    | # Kasra
                             ٍ    | # Tanwin Kasr
                             ْ    | # Sukun
                             ـ     # Tatwil/Kashida
                         """, re.VERBOSE)
    text = re.sub(noise, '', text)
    return text

def normalize_arabic(text):
    text = re.sub("[إأٱآا]", "ا", text, flags=re.MULTILINE)
    text = re.sub("ى", "ي", text, flags=re.UNICODE)
    text = re.sub("ؤ", "ء", text, flags=re.UNICODE)
    text = re.sub("ئ", "ء", text, flags=re.UNICODE)
    return text

def clean_tweet(text):
    URL_re = r'(http|https|ftp)://[a-zA-Z0-9\\./]+' # replace URLs
    HASH_re = r'#(\w+)' # replace Hashtags
    HANDLE_re = r'@(\w+)' # replace Handles
    REPUNC_re = r'[\?\.\!]+(?=[\?\.\!])' # normalize repeating punctuations (e.g., ?? or !!!)
    RECHAR_re = r'(\w)\1{1,}' # remove repeating charachters 
    NEWLINE_re = r'[\n]+' # remove new lines 
    NUM_CHARS_AR_re = r'[١٢٣٤٥٦٧٨٩٠؟!٪:،؛.]+'
    NUM_CHARS_EN_re = r'[0123456789!"$%&\\\'()*+,-.;<>=?@[]^_`{═}—“”•‹|~«°»˓ ̮ ̯]+'
    WHITE_SPACE_re = r'\s+'
    
    text = re.sub(URL_re,  r'', text, flags=re.UNICODE)
    text = re.sub(HASH_re,  r'', text, flags=re.UNICODE)
    text = re.sub(HANDLE_re,  r'', text, flags=re.UNICODE)
    text = re.sub(REPUNC_re,  r' ', text, flags=re.UNICODE)
    text = re.sub(RECHAR_re,  r'\1', text, flags=re.UNICODE)
    text = re.sub(NEWLINE_re,  r'', text, flags=re.UNICODE)
    text = re.sub(NUM_CHARS_AR_re,  r'', text, flags=re.UNICODE)
    text = re.sub(NUM_CHARS_EN_re,  r'', text, flags=re.UNICODE)
    text = re.sub(WHITE_SPACE_re,  r' ', text, flags=re.UNICODE)
    
    return text

tweets.text = tweets.text.apply(lambda x: denoise_arabic(str(x)))
tweets.text = tweets.text.apply(lambda x: normalize_arabic(str(x)))
tweets.text = tweets.text.apply(lambda x: clean_tweet(str(x)))
toremove = '! ... ? ! \u200b ...  ?  y ֆ ⤵ ⇜ ✎ ❃ ❥ ༺ ༻ ┄ ࿐ | ᅠ\u200b \u200c \u200d — “ ” • ‹ \u2066 \u2067 \u2069 ⃣ ↓ ↴ ⇣ ⌝ ┈ ┉ ┊ ═ ! " # $ % & \' ( ) * + , - ... / 0 1 10 10/10 12 13 1373 14 140جرام 15 16 17 17,0 18 19 1935 2 20 2015 2017 2020 2030 21 24 25 26 27 28 2leh_q 3 30 36 4 40 5 5/5 50 6 60 61 7 7/7 70 8 80 9 90 ; < = > ? @ [ ] ^ _ _20 ` { } ~ « ° » ˓ ̮ ̯'        
regex = re.compile("(" + "|".join(map(re.escape, toremove.split(' '))) + ")")
tweets.text = tweets.text.apply(lambda x: re.sub(regex, r'', x))
tweets.text = tweets.text.apply(lambda x: re.sub(r'[0123456789A-Za-z?!]', r'', x))
```

<div style="direction:rtl"> 
بعد أن انتهيت من تنظيف النصوص، اكتشفت أن خانة " المدينة" في عدد قليل جداً من الصفوف لم يكن صحيحا. لتسوية الأمر، سنقرأ جدول المدن ثم نفلتر كل الصفوف الخاطئة.
</div>


```python
cities = pd.read_csv('./data/cities_lat_long.csv')
tweets = tweets[tweets.city.isin(cities.city.tolist())]
tweets.reset_index(inplace=True, drop=['index'])
```

<div style="direction:rtl"> 
هذه هي قائمة المدن في قاعدة البيانات الحالية..
</div>


```python
tweets.city.unique()
```




    array(['Riyadh', 'Tabuk', 'Jeddah', 'Hail', 'Mecca', 'Medina', 'Jubail',
           'Hofuf', 'Jazan', 'Al-Kharj', 'Taif', 'Qatif', 'Dammam', 'Abha',
           'Khamis Mushait', 'Najran', 'Buraidah', 'Yanbu', 'Khobar'], dtype=object)



<div style="direction:rtl; font-size:25px; text-align:right; font-weight: bold;" > 
تحليل البيانات الاسكتشافي (Exploratory Data Analysis)
</div>

<div style="direction:rtl"> 
نبدأ الآن استكشاف البيانات. استكشاف البيانات هي عملية إبداعية وفضولية في المقام الأول، إذا لا يوجد لها أي قواعد تحكمها ولا مبادئ تقف عندها. الهدف هو أن نفهم بنية البيانات وطبيعتها بشكل أفضل. تقوم هذه المرحلة بشكل أساسي على التساؤلات التي نود الإجابة عنها. أدناه سردت أبرز الأسئلة التي طالما كانت حاضرة في ذهني ودفعتني للقيام بهذا المشروع..
<br>
<ul>
<li>ماهي الرموز التعبيرية الأكثر استخداماً في جميع المدن وماهي الرموز التعبيرية الأقل شيوعاً؟ وهل تشبه هذه القائمة موقع <a href="http://emojitracker.com">EmojiTracker</a> المختص برصد الوجوه التعبيرية الأكثر استخداماً في موقع تويتر؟</li>
<li>ماهي المدن الأكثر استخداماً للرموز التعبيرية وماهي المدن الأقل استخداماً للرموز التعبيرية؟</li>
<li>هل هناك رموز تعبيرية محددة تستخدم بشكل أكثر في كل مدينة؟ مثلاً هل هناك رموز تعبيرية يستخدمها سكان جدة بنسبة أعلى من سكان الدمام؟</li>
<li>هناك رموز تعبيرية "إيجابية" مثل القلب والورد والوجه الضاحك، بينما هناك رموز تعبيرية سلبية مثل القلب المكسور والوجه الباكي وغيرها. هل نستطيع معرفة أي المدن أكثر استخداماً للرموز الإيجابية وأيها أكثر استخداماً للرموز السلبية؟</li>
<li>أيضاً، بما أن لدينا معلومات حول وقت كتابة التغريدات، هل حديث الليل يشبه حديث النهار؟</li>
</ul>
<br>
لا أحب أن أستطرد في الأسئلة التي يمكننا البحث عنها حيث أن هذه البيانات غنية وثرية جداً. كل هذه الأسئلة ونحن لم نستخدم بعد نصوص التغريدات أو علاقات المغردين ببعضهم البعض وغير ذلك من الأسئلة التي تصلح لمشاريع قادمة. في هذا المشروع سوف نركز فقط على أنماط استخدام الرموز التعبيرية بين مجموعة من مدن المملكة..
<br/> <br/>
الآن سنبدأ بتجهيز البيانات بشكل مناسب للإجابة عن الأسئلة. بكل بساطة، سنجهز جدولاً آخر للتغريدات بحيث تبقى الصفوف تمثل التغريدات ولكن الأعمدة هذه المرة تمثل الرموز التعبيرية. وكل رمز تعبيري يستخدم سيكون له قيمة 1 فيما ستكون الباقية 0. ومن هذا الجدول نستطيع المضي قدماً في التحليل. 

</div>


```python
emojis_to_extract = '☀ ☁ ☄ ★ ☎ ☔ ☕ ☘ ☝ ☹ ☺ ☻ ♀ ♂ ♠ ♡ ♥ ♨ ♩ ♪ ⚕ ⚘ ⚜ ⚡ ⚫ ⚽ ⛅ ✅ ✈ ✉ ✋ ✌ ✍ ✔ ✨ ✿ ❀ ❁ ❃ ❄ ❋ ❌ ❕ ❗ ❣ ❤ ❥ ⤵ ⬇ ⭐ ️ 🇦 🇧 🇨 🇪 🇬 🇭 🇮 🇰 🇱 🇲 🇳 🇵 🇶 🇷 🇸 🇹 🇺 🇼 🇾 🌈 🌌 🌎 🌑 🌕 🌙 🌚 🌝 🌞 🌟 🌤 🌥 🌧 🌨 🌬 🌱 🌲 🌴 🌷 🌸 🌹 🌺 🌻 🌼 🌾 🌿 🍀 🍁 🍂 🍃 🍆 🍇 🍌 🍏 🍑 🍒 🍔 🍥 🍦 🍫 🍭 🍰 🍳 🍺 🍻 🍼 🎀 🎁 🎂 🎈 🎉 🎊 🎓 🎤 🎧 🎨 🎬 🎭 🎵 🎶 🎷 🎹 🎻 🎼 🏁 🏃 🏆 🏳 🏹 🏻 🏼 🏽 🏾 🏿 🐒 🐙 🐥 🐸 👀 👄 👅 👆 👇 👈 👉 👊 👋 👌 👍 👎 👏 👐 👑 👞 👧 👨 👩 👪 👭 👰 👵 👶 👸 👻 👼 👽 💀 💁 💃 💅 💆 💉 💋 💌 💍 💎 💐 💓 💔 💕 💖 💗 💘 💙 💚 💛 💜 💝 💞 💟 💡 💢 💣 💤 💥 💦 💧 💨 💩 💪 💫 💬 💭 💯 💰 💵 💸 📌 📍 📖 📚 📝 📞 📣 📩 📮 📯 📱 📲 📷 📻 🔁 🔐 🔑 🔕 🔗 🔝 🔞 🔥 🔪 🔫 🔴 🔵 🔶 🔹 🔻 🕊 🕋 🕑 🕛 🕞 🕺 🖐 🖒 🖕 🖖 🖤 🗓 🗝 🗣 😀 😁 😂 😃 😄 😅 😆 😇 😈 😉 😊 😋 😌 😍 😎 😏 😐 😑 😒 😓 😔 😕 😖 😗 😘 😙 😚 😛 😜 😝 😞 😟 😠 😡 😢 😣 😤 😥 😦 😧 😨 😩 😪 😫 😬 😭 😮 😯 😰 😱 😲 😳 😴 😶 😷 😹 😻 😼 😽 😾 😿 🙀 🙁 🙂 🙃 🙄 🙅 🙆 🙇 🙈 🙉 🙊 🙋 🙌 🙍 🙏 🚀 🚗 🚫 🚬 🚶 🛑 🤐 🤒 🤓 🤔 🤕 🤗 🤘 🤙 🤚 🤛 🤝 🤞 🤢 🤣 🤤 🤦 🤧 🤷 🥀 🦋'
emojis_to_extract = emojis_to_extract.split(' ')
```


```python
word_vectorizer_count = CountVectorizer(ngram_range=(1, 1),
                                        strip_accents='unicode', 
                                        tokenizer=nltk.tokenize.TweetTokenizer().tokenize,
                                        vocabulary=emojis_to_extract,
                                       )
```


```python
%%time
text_vectors = word_vectorizer_count.fit_transform(tweets.text)
print(text_vectors.shape)
```

    (134763, 362)
    CPU times: user 11.9 s, sys: 71.2 ms, total: 11.9 s
    Wall time: 15.1 s



```python
features = np.array(word_vectorizer_count.get_feature_names())
```

<div style="direction:rtl"> 
لابد أن نتأكد أن الجدول الجديد مبني بشكل صحيح.. سنطبع تغريدة عشوائية أولاً، ثم نرى ماهي الرموز التعبيرية التي تساوي 1 لذات التغريدة. يجب أن يكون الاثنان متساويان.
</div>


```python
idx = np.random.choice(tweets.shape[0], 1)[0]
print('Tweet:\t\t', tweets.text[idx], 
      '\nFeatures:\t\t', features[np.where(text_vectors.todense()[idx]>0)[1]]
     )
```

    Tweet:		  اذا لقيتي عطيني 🤗💔 
    Features:		 ['💔' '🤗']



```python
idx = np.random.choice(tweets.shape[0], 1)[0]
print('Tweet:\t\t', tweets.text[idx], 
      '\nFeatures:\t\t', features[np.where(text_vectors.todense()[idx]>0)[1]]
     )
```

    Tweet:		 جد 😍😍😍💙💙 ياسلام 😍😍👏 خلص ولا لسا  
    Features:		 ['👏' '💙' '😍']



```python
idx = np.random.choice(tweets.shape[0], 1)[0]
print('Tweet:\t\t', tweets.text[idx], 
      '\nFeatures:\t\t', features[np.where(text_vectors.todense()[idx]>0)[1]]
     )
```

    Tweet:		  نتفتي الحلوه فيها كياته موطبيعي 😭❤❤🍥  
    Features:		 ['❤' '🍥' '😭']


<div style="direction:rtl"> 
يبدو أن الأمور على ما يرام (ماعدا بعض الأخطاء المتعلقة بترميز النصوص وليس بسبب أداءنا للتحليل).
</div>


```python
emoj_df = pd.DataFrame(text_vectors.todense(), columns=features)
```


```python
emoj_df.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>☀</th>
      <th>☁</th>
      <th>☄</th>
      <th>★</th>
      <th>☎</th>
      <th>☔</th>
      <th>☕</th>
      <th>☘</th>
      <th>☝</th>
      <th>☹</th>
      <th>...</th>
      <th>🤝</th>
      <th>🤞</th>
      <th>🤢</th>
      <th>🤣</th>
      <th>🤤</th>
      <th>🤦</th>
      <th>🤧</th>
      <th>🤷</th>
      <th>🥀</th>
      <th>🦋</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 362 columns</p>
</div>




```python
tweets_emoj = pd.concat([tweets.city, emoj_df], axis=1, ignore_index=False)
```


```python
tweets_emoj.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>☀</th>
      <th>☁</th>
      <th>☄</th>
      <th>★</th>
      <th>☎</th>
      <th>☔</th>
      <th>☕</th>
      <th>☘</th>
      <th>☝</th>
      <th>...</th>
      <th>🤝</th>
      <th>🤞</th>
      <th>🤢</th>
      <th>🤣</th>
      <th>🤤</th>
      <th>🤦</th>
      <th>🤧</th>
      <th>🤷</th>
      <th>🥀</th>
      <th>🦋</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Riyadh</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Riyadh</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Riyadh</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Riyadh</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Riyadh</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 363 columns</p>
</div>



<div style="direction:rtl"> 
بعد أن تم بناء الجدول بشكل سليم وإضافة خانة المدينة، سنقوم الآن بالخطوة النهائية وهي تجميع الصفوف لتصبح المدينة في خانة الصفوف والرموز التعبيرية في خانة الأعمدة. الأرقام الآن ستمثل مجموع المرات التي استخدم فيها رمز تعبيري ما في التغريدات القادمة من مدينة ما. طبعت رأس الجدول لنرى كيف يبدو ذلك.
</div>


```python
tweets_emoj_city = tweets_emoj.groupby(['city']).agg(np.sum)
```


```python
tweets_emoj_city.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>☀</th>
      <th>☁</th>
      <th>☄</th>
      <th>★</th>
      <th>☎</th>
      <th>☔</th>
      <th>☕</th>
      <th>☘</th>
      <th>☝</th>
      <th>☹</th>
      <th>...</th>
      <th>🤝</th>
      <th>🤞</th>
      <th>🤢</th>
      <th>🤣</th>
      <th>🤤</th>
      <th>🤦</th>
      <th>🤧</th>
      <th>🤷</th>
      <th>🥀</th>
      <th>🦋</th>
    </tr>
    <tr>
      <th>city</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Abha</th>
      <td>4</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>12</td>
      <td>0</td>
      <td>3</td>
      <td>222</td>
      <td>...</td>
      <td>2</td>
      <td>0</td>
      <td>7</td>
      <td>23</td>
      <td>5</td>
      <td>97</td>
      <td>1</td>
      <td>0</td>
      <td>18</td>
      <td>3</td>
    </tr>
    <tr>
      <th>Al-Kharj</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>47</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>8</td>
      <td>18</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Buraidah</th>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>10</td>
      <td>4</td>
      <td>5</td>
      <td>116</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>23</td>
      <td>3</td>
      <td>24</td>
      <td>2</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Dammam</th>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>12</td>
      <td>4</td>
      <td>3</td>
      <td>182</td>
      <td>...</td>
      <td>2</td>
      <td>3</td>
      <td>3</td>
      <td>101</td>
      <td>8</td>
      <td>167</td>
      <td>1</td>
      <td>4</td>
      <td>14</td>
      <td>2</td>
    </tr>
    <tr>
      <th>Hail</th>
      <td>2</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>16</td>
      <td>1</td>
      <td>1</td>
      <td>149</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>3</td>
      <td>22</td>
      <td>4</td>
      <td>88</td>
      <td>0</td>
      <td>8</td>
      <td>15</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 362 columns</p>
</div>



<div style="direction:rtl"> 
قبل أن نبدأ أي تحليل (نعم لم نحلل شيئاً بعد!)، من الوضح أن لدينا مشكلة في توزيع استخدامات الرموز التعبيرية بين المدن. فمدينة كالرياض يوجد بها عدد هائل من المغردين وعدد هائل من التغريدات كذلك ومن غير المنطقي مقارنتها مع مدينة نجران مثلاً بدون احتساب التفاوت في عدد التغريدات وعدد المغردين.

<br/> <br/>

لحل هذا الإشكال، سنقوم بتغيير الأرقام لتعكس النسب (داخل المدينة) وذلك بقسمة عدد مرات استخدام أي رمز تعبيري ( مثلاً الوجه الضاحك) داخل مدينة ما بعدد الوجود التعبيرية في تغريدات تلك المدينة. بعد هذه الخطوة نستطيع أن نقارن بين المدن لنقول مثلا أن نسبة استخدام الوجه الضاحك داخل الرياض أكثر من نسبة استخدام الوجه الضاحك داخل نجران بالرغم من اختلاف عدد التغريدات من كل مدينة.
</div>


```python
tweets_emoj_city_size = tweets_emoj.groupby(['city']).size()
```


```python
tweets_emoj_city_ratio = np.divide(tweets_emoj_city, np.sqrt(tweets_emoj_city_size[:,None]))
```

<div style="direction:rtl"> 
أدناه شكل الجدول النهائي والذي يتكون الآن من نسب بدلاً من أرقام صحيحة.

<br />

للإجابة عن سؤالنا الأول، نستطيع مباشرة أن نحتسب "متوسط" استخدام الرموز التعبيرية داخل كل مدينة وثم نرتب النتائج تصاعدياً لنرى أي المدن لديها أكبر عدد من استخدام الرموز التعبيرية (نهاية القائمة) وأي المدن لديها أقل عدد من الرموز التعبيرية (بداية القائمة).

</div>


```python
tweets_emoj_city_ratio.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>☀</th>
      <th>☁</th>
      <th>☄</th>
      <th>★</th>
      <th>☎</th>
      <th>☔</th>
      <th>☕</th>
      <th>☘</th>
      <th>☝</th>
      <th>☹</th>
      <th>...</th>
      <th>🤝</th>
      <th>🤞</th>
      <th>🤢</th>
      <th>🤣</th>
      <th>🤤</th>
      <th>🤦</th>
      <th>🤧</th>
      <th>🤷</th>
      <th>🥀</th>
      <th>🦋</th>
    </tr>
    <tr>
      <th>city</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Abha</th>
      <td>0.041536</td>
      <td>0.020768</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.041536</td>
      <td>0.124609</td>
      <td>0.000000</td>
      <td>0.031152</td>
      <td>2.305257</td>
      <td>...</td>
      <td>0.020768</td>
      <td>0.000000</td>
      <td>0.072688</td>
      <td>0.238833</td>
      <td>0.051920</td>
      <td>1.007252</td>
      <td>0.010384</td>
      <td>0.000000</td>
      <td>0.186913</td>
      <td>0.031152</td>
    </tr>
    <tr>
      <th>Al-Kharj</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.040639</td>
      <td>0.000000</td>
      <td>0.020319</td>
      <td>0.955016</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.020319</td>
      <td>0.000000</td>
      <td>0.020319</td>
      <td>0.162556</td>
      <td>0.365751</td>
      <td>0.000000</td>
      <td>0.020319</td>
      <td>0.060958</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Buraidah</th>
      <td>0.012537</td>
      <td>0.037612</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.125373</td>
      <td>0.050149</td>
      <td>0.062686</td>
      <td>1.454324</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.012537</td>
      <td>0.288357</td>
      <td>0.037612</td>
      <td>0.300895</td>
      <td>0.025075</td>
      <td>0.012537</td>
      <td>0.050149</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>Dammam</th>
      <td>0.008719</td>
      <td>0.026157</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.043595</td>
      <td>0.000000</td>
      <td>0.104629</td>
      <td>0.034876</td>
      <td>0.026157</td>
      <td>1.586874</td>
      <td>...</td>
      <td>0.017438</td>
      <td>0.026157</td>
      <td>0.026157</td>
      <td>0.880628</td>
      <td>0.069753</td>
      <td>1.456088</td>
      <td>0.008719</td>
      <td>0.034876</td>
      <td>0.122067</td>
      <td>0.017438</td>
    </tr>
    <tr>
      <th>Hail</th>
      <td>0.018734</td>
      <td>0.046835</td>
      <td>0.009367</td>
      <td>0.009367</td>
      <td>0.000000</td>
      <td>0.009367</td>
      <td>0.149873</td>
      <td>0.009367</td>
      <td>0.009367</td>
      <td>1.395697</td>
      <td>...</td>
      <td>0.000000</td>
      <td>0.009367</td>
      <td>0.028101</td>
      <td>0.206076</td>
      <td>0.037468</td>
      <td>0.824304</td>
      <td>0.000000</td>
      <td>0.074937</td>
      <td>0.140506</td>
      <td>0.037468</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 362 columns</p>
</div>




```python
tweets_emoj_city_ratio.mean(axis=1).sort_values()
```




    city
    Najran            0.177028
    Jazan             0.182721
    Khamis Mushait    0.251193
    Al-Kharj          0.312089
    Yanbu             0.333752
    Taif              0.360855
    Jubail            0.388580
    Hofuf             0.410994
    Tabuk             0.448958
    Buraidah          0.492625
    Khobar            0.649636
    Abha              0.668996
    Hail              0.695209
    Dammam            0.713978
    Riyadh            0.723166
    Medina            0.732782
    Jeddah            0.736642
    Mecca             0.748855
    Qatif             0.826966
    dtype: float64



<div style="direction:rtl"> 
كما نستطيع عمل ذات الشيء مع الرموز التعبيرية. وهو أخذ متوسط نسب استخدام كل رمز تعبيري في جميع المدن لنعرف ماهي الرموز التعبيرية الأكثر استخداماً
</div>


```python
tweets_emoj_city_ratio.mean(axis=0).sort_values()[::-1][:10]
```




    😂    29.799377
    ️    20.908354
    ❤    18.033336
    💔    13.382810
    😭    10.055071
    💙     9.735467
    😍     5.278099
    🏻     3.945968
    🌹     3.881957
    💛     3.713981
    dtype: float64



<div style="direction:rtl"> 
أو الرموز التعبيرية الأقل استخداماً...
</div>


```python
tweets_emoj_city_ratio.mean(axis=0).sort_values()[:10]
```




    🛑    0.000000
    🕞    0.000000
    🕑    0.000000
    🍏    0.000462
    🕛    0.001069
    🍒    0.002823
    🍇    0.003647
    🍌    0.004194
    🔑    0.004369
    🍑    0.004648
    dtype: float64



<div style="direction:rtl"> 
كما نستطيع تمثيل هذه البيانات في شكل يسهل فهمه فيما يعرف ب "الخرائط الحرارية". بكل بساطة، هي إعادة رسم لذات الجدول بنفس الشكل، ولكن الأرقام الآن يتم تحويلها إلى ألوان مأخوذة من طيف لوني يجعل من السهل رؤية القيم العالية والقيم المتدنية.
</div>


```python
fig, ax = plt.subplots(figsize=(15,5))
sns.heatmap(tweets_emoj_city_ratio, ax=ax)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11aab2400>




![png](output_51_1.png)


<div style="direction:rtl"> 
يبدو أن لدينا مشكلة شبيهة في الرموز التعبيرية فيما يخص اختلاف توزيع الاستخدام. فالوجه الضاحك مثلاً يحتل المركز الأول وبالتالي ستكون نسبته عالية في أغلب المدن (وهو بالمناسبة أحد الخطوط العمودية الداكنة في الشكل أعلاه). وهذا لن يجعلنا نعرف إن كانت مدينة ما تستخدمه بشكل أكثر من مدن أخرى (بعد أخذ كل العوامل الأخرى بالاعتبار). فما الحل مع الرموز التعبيرية ؟

<br> <br>

الحل بسيط جداً. فكر مثلاً كيف سنقارن بين رمز الوجه الضاحك ورمز الفتاة الراقصة. نحن نعرف أن "متوسط" استخدام الرمزين مختلف أساساً وبالتالي لكل رمز توزيع مختلف بين المدن. أبسط حل هو أن نقوم بتثبيت هذا المتوسط عند قيمة محددة لكل الرمزين لنتمكن بعدها من المقارنة العادلة. هذا بلغة الإحصاء يسمى 
<a href="https://en.wikipedia.org/wiki/Normalization_(statistics)">Normalization</a> وأبسط أنواعه يسمى z-scoring ويعني طرح قيمة المتوسط من قيم التوزيع ليصبح المتوسط الجديد هو صفر، ثم قسمة قيم التوزيع على الانحراف المعياري ليصبح الانحراف المعياري موحداً بين الرموز كذلك.

</div>


```python
df_scaled = (tweets_emoj_city_ratio - tweets_emoj_city_ratio.mean(axis=0)) / tweets_emoj_city_ratio.std(axis=0)
```

<div style="direction:rtl"> 
وكما هو واضح من الشكل التالي، نستطيع أن نرى الأنماط المختلفة لكل رمز تعبيري بشكل أوضح. 

خذ بالاعتبار أن متوسط استخدام كل رمز تعبيري هو صفر. بالتالي، حينما تأخذ أي مدينة (لنأخذ أبها مثلاً)، ستجد أن القيم الموجبة (النقاط الحمراء) تمثل معدل استخدام أعلى من باقي الرموز التعبيرية، فيما القيم السالبة (النقاط الزرقاء) تمثل معدل استخدام أقل من باقي الرموز التعبيرية.

</div>


```python
fig, ax = plt.subplots(figsize=(15,5))
sns.heatmap(df_scaled, ax=ax)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11be20e10>




![png](output_55_1.png)


<div style="direction:rtl"> 
بعد هذه الخطوة ، نحن على أتم الجاهزية للإجابة عن أبرز الوجوه التعبيرية المستخدمة التي تميز مدينة ما عن المدن الأخرى وهذا ما سنراه لكل المدن.
</div>


```python
print('')
df_scaled = df_scaled.fillna(0)
for city in tweets.city.unique():
    print(city,':', ' '.join(df_scaled.loc[city].sort_values()[::-1].index[:15].tolist()))
    print('\n')
```

    
    Riyadh : 🍏 📞 🌲 🍻 😼 🎤 🙀 ❀ ♩ 🎁 📚 🍔 🍌 📲 🚬
    
    
    Tabuk : ★ 🤓 💟 🍑 🤣 💍 🌿 ⛅ ❌ 😑 📮 💦 😱 💢 🍳
    
    
    Jeddah : 🇺 🌟 🍀 🍼 💸 👅 💚 🍺 👶 ❋ 🌌 💢 🎉 😣 🙉
    
    
    Hail : 👪 🌙 🇮 ⚡ ❣ 🇹 🤙 🔻 ⚜ 🎬 🏹 📖 💤 🍇 🇲
    
    
    Mecca : 🕋 ❥ 🖐 ☝ 👻 ✿ 👼 ⚘ 🕺 😮 👭 💬 🙅 😻 😗
    
    
    Medina : 🏁 ⤵ 💪 🔵 🖒 🤕 🍃 💡 🇬 💨 🗓 😛 🌺 👌 💫
    
    
    Jubail : 🎧 👞 ✅ 🤛 👄 🇶 💝 👶 🌬 ❀ 🌤 🙃 👉 ⚕ 👭
    
    
    Hofuf : 👵 👰 🇧 🌥 🇭 🙍 🌴 ❃ ❁ 🍒 👑 🗝 📌 ☺ 💆
    
    
    Jazan : 😨 🌤 ✉ 🇳 🎹 ⛅ 🙅 🌱 🙇 🖖 💢 🙀 ⚘ 😹 🇮
    
    
    Al-Kharj : 🕛 😲 🤤 💝 ♪ 📝 😰 😹 ❁ ⬇ 😱 🍺 💍 💥 🗝
    
    
    Taif : 🎭 🔗 ❄ 🌨 💐 🔶 💥 ⭐ 🦋 ☔ 🌧 💀 🙁 👆 ⚽
    
    
    Qatif : 😭 🤷 🤝 🖕 🏿 😞 😂 😡 🙂 😰 🙇 ♀ 🙆 🤦 👎
    
    
    Dammam : 📱 🇰 🇼 🗣 🙌 👊 🍫 🎓 ⬇ 🍰 😀 🌱 💦 ✔ 🌼
    
    
    Abha : 🐙 🍆 🍭 🎻 📯 🇨 💖 🏾 🔞 🎵 🏆 ☔ 🚶 🍰 🌞
    
    
    Khamis Mushait : 🍥 🎀 💎 🔫 💞 💋 🌌 🙉 💥 😫 😘 😠 📲 👨 🎻
    
    
    Najran : 🇵 🇱 🎊 ✉ 😆 🇷 🎈 📮 😝 🎹 🏃 🎷 ❕ ♪ 🌴
    
    
    Buraidah : 🔝 🔁 ♠ ⚫ 📣 🔹 🌑 🇾 🌕 😙 😿 ❕ 🐥 🍁 🇪
    
    
    Yanbu : 🏳 😖 😾 💧 💩 🔑 🐒 🔪 ☻ 😈 🌈 🖒 🏹 😴 ⚽
    
    
    Khobar : 😯 ♨ 💉 ☀ ☎ 👧 🌝 🚗 😤 😥 🌈 👩 😦 🤧 🤢
    
    


<div style="direction:rtl"> 

بناء على الشكل أعلاه سأذكر انطباعي بخصوص هذه الطريقة لحساب الرموز التعبيرية التي تميز كل مدينة عن أخرى. أعتقد أن هناك طرق أكثر متانة وثقة من هذه الطريقة البسيطة مثل 
<a href="https://en.wikipedia.org/wiki/Correspondence_analysis">Correspondence Analysis</a> أو بناء جدول عدد الرموز أعلاه بطريقة مختلفة مثل استخدام خوارزمية 
<a href="https://en.wikipedia.org/wiki/Tf%E2%80%93idf">TF-IDF</a> بدلاً من العدد.

كما ترى، فهذه الطريقة أعلاه تثبت "المتوسط" بين المدن ليكون في الصفر ولكننا مازلنا نرى أن المدن الكبرى التي تمتلك نصيباً أكبر من التغريدات لديها نصيب الذهب من الرموز التعبيرية وذلك لوجود عدد كبير من القيم الموجبة لدى المدن الكبرى. هذا إما قد يعكس انحيازاً لعدد التغريدات، أو أن المغردين في المدن الكبرى يستخدمون الرموز التعبيرية بشكل أكبر من المغردين في باقي المدن. أميل شخصياً للتفسير اللاحق وذلك لأني أكاد أرى "منطقة حمراء" لكل مدينة أو مجموعة من الرموز التي تميزها عن غيرها على الرغم من قلة كمية التغريدات فيه (شاهد الشكل أعلاه). على أية حال، هذه الطريقة مازالت مفيدة لمعرفة أي الرموز هي الأكثر استخداماً من غير أن نكترث للفروقات بين الرمز الأول والثاني إن كان كبيراً أم ضئيلاً.

</div>

<div style="direction:rtl; font-size:25px; text-align:right; font-weight: bold;" > 
تحليل المشاعر (Sentiment Analysis)
</div>

<div style="direction:rtl"> 
الآن سننتقل للسؤال التالي، وهو حول المحتوى العاطفي للرموز التعبيرية والذي يتراوح من رموز تعبيرية إيجابية (كالقلب أو الوجه الضاحك) إلى رموز تعبيرية سلبية (كالوجه الباكي أو الوجه الغاضب). قد تبدو هذه مهمة صعبة، إذا كيف نحدد "قيمة عاطفية" لكل وجه تعبيري؟ لحسن الحظ، هناك من سبق وقام بهذه المهمة. ففي 
<a href="http://journals.plos.org/plosone/article/file?id=10.1371/journal.pone.0144296&type=printable">ورقة علمية</a>
منشورة في مجلة PLOS One، استخدم الباحثون حوالي مليون تغريدة من أكثر من 13 لغة أوروبية وأكثر من 82 محلل ليخرجوا ببحث ممتع كان هذا المعجم التعبيري - العاطفي أبرز نتائجه. لاحظ أنه قد يكون هناك اختلافات ثقافية طفيفة بين الرموز التعبيرية في التغريدات الأوروبية (سواء بالإنجليزية أو بغيرها) إلا أني أعتقد أن هذه المسائل طفيفة عملياً إذا ما كان الغرض هو المعرفة بشكل عام وليس تصنيف كل تغريدة على حدة مثلاً.

<br><br>
سبق وأن نسخت 
<a href="http://kt.ijs.si/data/Emoji_sentiment_ranking/index.html">الجدول المرفق هنا</a>
من الورقة ليصبح في جدول مستقل قابل للقراءة والاستخدام داخل بايثون.

<br><br>

في هذا الجدول، نجد أن لكل رمز تعبيري "درجة عاطفية" تتراوح من +1 إلى -1، أو من موجب إلى سالب (فيما الصفر يعتبر محايد). سنقوم بقراءة كل تغريدة واستخراج كل الرموز التعبيرية منها، ثم حساب متوسط "الدرجة العاطفية" للتغريدة بناء على الرموز المستخدمة (قد يجادلني البعض في هذا التوجه، ولكن هذا هو الطريق الأبسط على أية حال).

</div>


```python
senti = pd.read_csv('./data/sentiment_scores.csv')
```


```python
senti.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Char</th>
      <th>Image</th>
      <th>Unicodecodepoint</th>
      <th>Occurrences</th>
      <th>Position</th>
      <th>Neg</th>
      <th>Neut</th>
      <th>Pos</th>
      <th>Sentiment</th>
      <th>bar</th>
      <th>name</th>
      <th>block</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>😂</td>
      <td>😂</td>
      <td>0x1f602</td>
      <td>14622.0</td>
      <td>0.805</td>
      <td>0.247</td>
      <td>0.285</td>
      <td>0.468</td>
      <td>0.221</td>
      <td></td>
      <td>FACE WITH TEARS OF JOY</td>
      <td>Emoticons</td>
    </tr>
    <tr>
      <th>1</th>
      <td>❤</td>
      <td>❤</td>
      <td>0x2764</td>
      <td>8050.0</td>
      <td>0.747</td>
      <td>0.044</td>
      <td>0.166</td>
      <td>0.790</td>
      <td>0.746</td>
      <td></td>
      <td>HEAVY BLACK HEART</td>
      <td>Dingbats</td>
    </tr>
    <tr>
      <th>2</th>
      <td>♥</td>
      <td>♥</td>
      <td>0x2665</td>
      <td>7144.0</td>
      <td>0.754</td>
      <td>0.035</td>
      <td>0.272</td>
      <td>0.693</td>
      <td>0.657</td>
      <td></td>
      <td>BLACK HEART SUIT</td>
      <td>Miscellaneous Symbols</td>
    </tr>
    <tr>
      <th>3</th>
      <td>😍</td>
      <td>😍</td>
      <td>0x1f60d</td>
      <td>6359.0</td>
      <td>0.765</td>
      <td>0.052</td>
      <td>0.219</td>
      <td>0.729</td>
      <td>0.678</td>
      <td></td>
      <td>SMILING FACE WITH HEART-SHAPED EYES</td>
      <td>Emoticons</td>
    </tr>
    <tr>
      <th>4</th>
      <td>😭</td>
      <td>😭</td>
      <td>0x1f62d</td>
      <td>5526.0</td>
      <td>0.803</td>
      <td>0.436</td>
      <td>0.220</td>
      <td>0.343</td>
      <td>-0.093</td>
      <td></td>
      <td>LOUDLY CRYING FACE</td>
      <td>Emoticons</td>
    </tr>
  </tbody>
</table>
</div>




```python
tweets['score'] = tweets.text.apply(lambda x: senti.loc[[emoji in x for emoji in senti['Char']]]['Sentiment'].mean())
```

<div style="direction:rtl"> 
الآن، بعد أن صار لكل تغريدة "درجة عاطفية"، نستطيع أن نجري حساباً بسيطة لمعرفة ترتيب المدن في مدى إيجابية/سلبية الرموز التعبيرية المستخدمة فيها. كما نلاحظ في الشكل القادم، مدينة حائل تحتل الأولى في استخدام الرموز الإيجابية فيما مدينة القطيف تعتبر الأخيرة.
</div>


```python
tweets.groupby(['city']).agg({'score':np.mean}).sort_values(by='score').plot(kind='bar', figsize=(15,5))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11be06be0>




![png](output_65_1.png)


<div style="direction:rtl"> 
وأخيراً، سنقوم باستخدام "الوقت" لمعرفة اختلاف أنماط استخدام الرموز الإيجابية أو السلبية. الشكل التالي يتكون من عدة أشكال مصغرة، شكل لكل مدينة. على محور السينات لدينا ساعات اليوم (من الساعة 12 صباحا في أقصى اليسار إلى منتصف الليل  يمينا)، فيما على محور الصادات لدينا ترمومتر المشاعر في التغريدات المستخرجة من هذه المدينة.

<br><br>
قارن مثلاً جدة والرياض ( والتي تملك مزاجاً سيئاً في فترة الظهرية) وغيرها من المدن. أغلب المدن تزيد القيمة فيها عن 0.3 وهي ذات القيمة في (الشكل 4) من الورقة سابقة الذكر حينما حسبوا متوسط قيمة المشاعر لكل التغريدات. مما يعكس بشكل ما أن استخدام الرموز الإيجابية يغلب على استخدام الرموز السلبية.. فيما تبقى حائل مدينة "الروقان".

</div>


```python
tweets.created_at = pd.to_datetime(tweets.created_at)
```


```python
pivoted = tweets.pivot_table('score', 
                              index=tweets.created_at.dt.hour, 
                              columns=tweets.city, 
                              aggfunc=np.mean)
pivoted.index.name = 'Hour'
plo = pivoted.plot(subplots=True, layout=(-1, 4), sharex=False, sharey=True, figsize=(18, 20))
```


![png](output_68_0.png)


<div style="direction:rtl; font-size:25px; text-align:right; font-weight: bold;" >
الختام..
</div>

<div style="direction:rtl"> 

لا أود الإطالة كما لا أود مناقشة النتائج بشكل مفصل. بل أحب دائماً أن أترك خيوط تفسير مفتوحة لمن أراد أن يتعمق. بالطبع لا أريد لأي أحد أن يتعمق في تفسير هذه النتائج نظراً للكمية المحدودة (والمنحازة إلى حد كبير) من التغريدات. هذا لا يجعلها غير مفيدة بل هي نقطة بداية لمن أحب التعمق.

<br><br>
كان الهدف من هذه التدوينة التحليلية هي إشباع فضولي العلمي والنقدي من خلال العمل على تحليل البيانات. ولأني أعرف أن الكثير يعتيرهم ذات الفضول ولديهم رغبة كبيرة لإجراء تحليلات مشابهة فقد قررت أولاً استخدام Jupyter Notebooks لكتابة التدوينة، ثم مشاركة الملفات الأولية لتصبح الفرصة متاحة لدى الجميع لإجراء أي تحليلات مشابهة (من دون الحاجة لإبلاغي أو الاستئذان مني في استخدام البيانات).

<br><br>
أتمنى أن تكون قد راقت لكم هذه التدوينة وأشبعت فضولكم كذلك. وربما دفعت بك إلى التفكير في طرق إبداعية لاستخدام البيانات العامة لفهم المجتمع الذي نعيش فيه بشكل أكبر وأدق. إن كانت لديك أي فكرة من هذا القبيل فأرجو التواصل معي لمشاركة المقترحات والأفكار.

<br><br>

إلى أن نراكم في المرة القادمة.. 
<a href="https://twitter.com/fahd09">فهد</a>

</div>
