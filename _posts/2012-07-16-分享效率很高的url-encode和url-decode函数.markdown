---
layout: post
date:   2012-07-16 20:17
categories: c++
---

在网上发现的url-encode和url-decode函数，写得简洁、高效，拿出来分享：

{% highlight cpp %}
/* Converts a hex character to its integer value */  
char from_hex(char ch) {  
  return isdigit(ch) ? ch - '0' : tolower(ch) - 'a' + 10;  
}  
  
/* Converts an integer value to its hex character*/  
char to_hex(char code) {  
  static char hex[] = "0123456789abcdef";  
  return hex[code & 15];  
}  
  
/* Returns a url-encoded version of str */  
/* IMPORTANT: be sure to free() the returned string after use */  
char *url_encode(char *str) {  
  char *pstr = str, *buf = malloc(strlen(str) * 3 + 1), *pbuf = buf;  
  while (*pstr) {  
    if (isalnum(*pstr) || *pstr == '-' || *pstr == '_' || *pstr == '.' || *pstr == '~')   
      *pbuf++ = *pstr;  
    else if (*pstr == ' ')   
      *pbuf++ = '+';  
    else   
      *pbuf++ = '%', *pbuf++ = to_hex(*pstr >> 4), *pbuf++ = to_hex(*pstr & 15);  
    pstr++;  
  }  
  *pbuf = '\0';  
  return buf;  
}  
  
/* Returns a url-decoded version of str */  
/* IMPORTANT: be sure to free() the returned string after use */  
char *url_decode(char *str) {  
  char *pstr = str, *buf = malloc(strlen(str) + 1), *pbuf = buf;  
  while (*pstr) {  
    if (*pstr == '%') {  
      if (pstr[1] && pstr[2]) {  
        *pbuf++ = from_hex(pstr[1]) << 4 | from_hex(pstr[2]);  
        pstr += 2;  
      }  
    } else if (*pstr == '+') {   
      *pbuf++ = ' ';  
    } else {  
      *pbuf++ = *pstr;  
    }  
    pstr++;  
  }  
  *pbuf = '\0';  
  return buf;  
}  
{% endhighlight %}


作者 [侯振永][1]     
写于2012 年 7月 16日

[1]: https://zhenyonghou.github.io/