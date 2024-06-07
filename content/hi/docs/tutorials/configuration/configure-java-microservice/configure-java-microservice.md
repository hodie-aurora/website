---
title: "माइक्रोप्रोफाइल, कॉन्फिगमैप्स और सीक्रेट्स का उपयोग करके कॉन्फिगरेशन को बाह्यीकृत करना"
content_type: ट्यूटोरियल
weight: 10
---

<!-- overview -->

इस ट्यूटोरियल में आप सीखेंगे कि अपने माइक्रोसर्विस के कॉन्फ़िगरेशन को कैसे और क्यों बाह्यीकृत करना है। 
विशेष रूप से, आप सीखेंगे कि एनवायरमेंट वेरिएबल सेट करने के लिए कुबेरनेट्स कॉन्फिगमैप्स और सीक्रेट्स का 
उपयोग कैसे करें और फिर माइक्रोप्रोफाइल कॉन्फिग का उपयोग करके उनका उपभोग करें। 


## {{% heading "prerequisites" %}}

### कुबेरनेट्स कॉन्फ़िगमैप्स और सीक्रेट बनाना

कुबेरनेट्स में डॉकर कंटेनर के लिए एनवायरमेंट वेरिएबल सेट करने के कई तरीके हैं, जिनमें शामिल हैं: Dockerfile, 
kubernetes.yml, Kubernetes ConfigMaps, और Kubernetes Secrets। ट्यूटोरियल में, आप सीखेंगे कि अपने 
एनवायरमेंट वेरिएबल सेट करने के लिए कुबेरनेट्स कॉन्फिगमैप्स और कुबेरनेट्स सीक्रेट्स का उपयोग कैसे करें, जिनके वैल्यू
आपके माइक्रोसर्विसेज में इंजेक्ट किए जाएंगे। कॉन्फिगमैप्स और सीक्रेट्स का उपयोग करने का एक लाभ यह है कि 
उन्हें कई कंटेनरों में फिर से उपयोग किया जा सकता है, जिसमें विभिन्न कंटेनरों के लिए अलग-अलग एनवायरमेंट वेरिएबल
को सौंपा जाना भी शामिल है।

कॉन्फिगमैप्स एपीआई ऑब्जेक्ट हैं जो गैर-गोपनीय key-value जोड़े को संग्रहीत करते हैं। 
इंटरएक्टिव ट्यूटोरियल में आप सीखेंगे कि एप्लिकेशन के नाम को संग्रहीत करने के लिए 
कॉन्फिगमैप का उपयोग कैसे करना है। कॉन्फ़िगमैप्स के संबंध में अधिक जानकारी के लिए, 
आप [दस्तावेज़ यहाँ पा सकते हैं](/docs/tasks/configure-pod-container/configure-pod-configmap/))।

हालाँकि सीक्रेट्स का उपयोग भी key-value जोड़े को संग्रहीत करने के लिए किया जाता है, 
वे कॉन्फिगमैप्स से भिन्न होते हैं क्योंकि वे गोपनीय/संवेदनशील जानकारी के लिए होते हैं और Base64 एन्कोडिंग
का उपयोग करके संग्रहीत होते हैं। यह सीक्रेट को क्रेडेंशियल्स, keys और टोकन जैसी चीज़ों  को संग्रहीत करने
के लिए उपयुक्त विकल्प बनाता है, जिनमें से पहला काम आप इंटरैक्टिव ट्यूटोरियल में करेंगे। सीक्रेट के बारे 
में अधिक जानकारी के लिए, आप [दस्तावेज़ यहाँ पा सकते हैं](/docs/concepts/configuration/secret/)।

### कोड से कॉन्फ़िग को बाह्यीकृत करना

बाह्यीकृत एप्लिकेशन कॉन्फ़िगरेशन उपयोगी है क्योंकि कॉन्फ़िगरेशन आमतौर पर आपके वातावरण के आधार पर 
बदलता है। इसे पूरा करने के लिए, हम Java के Contexts and Dependency Injection (CDI) और माइक्रोप्रोफाइल 
कॉन्फ़िगरेशन का उपयोग करेंगे। माइक्रोप्रोफाइल कॉन्फिग माइक्रोप्रोफाइल की एक विशेषता है, जो क्लाउड-नेटिव 
माइक्रोसर्विसेज को विकसित करने और डेप्लॉय करने के लिए open Java प्रौद्योगिकियों का एक सेट है।

सीडीआई (CDI) एक स्टैंडर्ड तरीका है जो एप्लिकेशन में डिपेंडेंसी इंजेक्शन (dependency injection) को आसान बनाता है।
इसकी मदद से, एप्लिकेशन को अलग-अलग हिस्सों (beans) से मिलाकर बनाया जा सकता है जो एक-दूसरे से कम जुड़े होते हैं। 
इससे एप्लिकेशन को बनाना और सुधारना आसान हो जाता है। माइक्रोप्रोफाइल कॉन्फिग ऐप्स और माइक्रोसर्विसेज को एप्लिकेशन, 
रनटाइम और एनवायरमेंट सहित विभिन्न स्रोतों से कॉन्फिग के गुण प्राप्त करने का एक मानक तरीका प्रदान करता है। स्रोत की 
परिभाषित प्राथमिकता के आधार पर, गुणों को स्वचालित रूप से गुणों के एक सेट में संयोजित किया जाता है जिसे 
एप्लिकेशन एपीआई के माध्यम से एक्सेस कर सकता है। साथ में, सीडीआई और माइक्रोप्रोफाइल का उपयोग कुबेरनेट्स 
कॉन्फिगमैप्स और सीक्रेट्स से बाहरी रूप से प्रदान की गई संपत्तियों को पुनः प्राप्त करने और आपके एप्लिकेशन कोड 
में इंजेक्ट करने के लिए इंटरएक्टिव ट्यूटोरियल में किया जाएगा।

कई ओपन सोर्स फ्रेमवर्क और रनटाइम माइक्रोप्रोफाइल कॉन्फ़िगरेशन को लागू और समर्थ करते हैं। पूरे इंटरैक्टिव 
ट्यूटोरियल के दौरान, आप ओपन लिबर्टी का उपयोग करेंगे, जो क्लाउड-नेटिव ऐप्स और माइक्रोसर्विसेज को बनाने 
और चलाने के लिए एक फ्लेक्सिबल ओपन-सोर्स Java रनटाइम है। हालाँकि, इसके बजाय किसी भी माइक्रोप्रोफाइल 
संगत रनटाइम का उपयोग किया जा सकता है।

## {{% heading "objectives" %}}

* एक कुबेरनेट्स कॉन्फ़िगमैप और सीक्रेट बनाएं
* माइक्रोप्रोफाइल कॉन्फ़िगरेशन का उपयोग करके माइक्रोसर्विस कॉन्फ़िगरेशन इंजेक्ट करें
  <!-- lessoncontent -->
  
## उदाहरण: माइक्रोप्रोफाइल, कॉन्फिगमैप्स और सीक्रेट्स का उपयोग करके कॉन्फिगरेशन को बाह्यीकृत करना

[इंटरैक्टिव ट्यूटोरियल प्रारंभ करें](/docs/tutorials/configuration/configure-java-microservice/configure-java-microservice-interactive/)

