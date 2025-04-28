---
title: "React Native 基于 Metro进行分包"
date: 2025-04-28T15:35:18+08:00
draft: false
categories: ["移动端"]
tags: ["React Native"]
---

##  一、分包原理
### (1) Bundle文件结构及内容说明
React Native打包形成的Bundle文件的内容从上到下依次是：

**Polyfills**：定义基本的JS环境（如：`__d()`函数、`__r()`函数、`__DEV__` 变量等）

**Module定义**：使用__d()函数定义所有用到的模块，该函数为每个模块赋予了一个模块ID，模块之间的依赖关系都是通过这个ID进行关联的。

**Require调用**：使用__r()函数引用根模块。
```
var __BUNDLE_START_TIME__=this.nativePerformanceNow?nativePerformanceNow():Date.now(),__DEV__=false,process=this.process||{};process.env=process.env||{};process.env.NODE_ENV=process.env.NODE_ENV||"production";
!(function(r){"use strict";r.__r=o,r.__d=function(r,i,n){if(null!=e[i])return;var o={dependencyMap:n,factory:r,hasError:!1,importedAll:t,importedDefault:t,isInitialized:!1,publicModule:{exports:{}}};e[i]=o},r.__c=n,r.__registerSegment=function(r,e){s[r]=e};var e=n(),t={},i={}.hasOwnProperty;function n(){return e=Object.create(null)}function o(r){var t=r,i=e[t];return i&&i.isInitialized?i.publicModule.exports:d(t,i)}function l(r){var i=r;if(e[i]&&e[i].importedDefault!==t)return e[i].importedDefault;var n=o(i),l=n&&n.__esModule?n.default:n;return e[i].importedDefault=l}function u(r){var n=r;if(e[n]&&e[n].importedAll!==t)return e[n].importedAll;var l,u=o(n);if(u&&u.__esModule)l=u;else{if(l={},u)for(var a in u)i.call(u,a)&&(l[a]=u[a]);l.default=u}return e[n].importedAll=l}o.importDefault=l,o.importAll=u;var a=!1;function d(e,t){if(!a&&r.ErrorUtils){var i;a=!0;try{i=v(e,t)}catch(e){r.ErrorUtils.reportFatalError(e)}return a=!1,i}return v(e,t)}var c=16,f=65535;function p(r){return{segmentId:r>>>c,localId:r&f}}o.unpackModuleId=p,o.packModuleId=function(r){return(r.segmentId<<c)+r.localId};var s=[];function v(t,i){if(!i&&s.length>0){var n=p(t),a=n.segmentId,d=n.localId,c=s[a];null!=c&&(c(d),i=e[t])}var f=r.nativeRequire;if(!i&&f){var v=p(t),h=v.segmentId;f(v.localId,h),i=e[t]}if(!i)throw Error('Requiring unknown module "'+t+'".');if(i.hasError)throw m(t,i.error);i.isInitialized=!0;var I=i,g=I.factory,y=I.dependencyMap;try{var _=i.publicModule;return _.id=t,g(r,o,l,u,_,_.exports,y),i.factory=void 0,i.dependencyMap=void 0,_.exports}catch(r){throw i.hasError=!0,i.error=r,i.isInitialized=!1,i.publicModule.exports=void 0,r}}function m(r,e){return Error('Requiring module "'+r+'", which threw an exception: '+e)}})('undefined'!=typeof globalThis?globalThis:'undefined'!=typeof global?global:'undefined'!=typeof window?window:this);
!(function(n){var e=(function(){function n(n,e){return n}function e(n){var e={};return n.forEach(function(n,r){e[n]=!0}),e}function r(n,r,l){if(n.formatValueCalls++,n.formatValueCalls>200)return"[TOO BIG formatValueCalls "+n.formatValueCalls+" exceeded limit of 200]";var f=t(n,r);if(f)return f;var c=Object.keys(r),g=e(c);if(d(r)&&(c.indexOf('message')>=0||c.indexOf('description')>=0))return o(r);if(0===c.length){if(h(r)){var p=r.name?': '+r.name:'';return n.stylize('[Function'+p+']','special')}if(s(r))return n.stylize(RegExp.prototype.toString.call(r),'regexp');if(y(r))return n.stylize(Date.prototype.toString.call(r),'date');if(d(r))return o(r)}var v,b,m='',j=!1,z=['{','}'];(v=r,Array.isArray(v)&&(j=!0,z=['[',']']),h(r))&&(m=' [Function'+(r.name?': '+r.name:'')+']');return s(r)&&(m=' '+RegExp.prototype.toString.call(r)),y(r)&&(m=' '+Date.prototype.toUTCString.call(r)),d(r)&&(m=' '+o(r)),0!==c.length||j&&0!=r.length?l<0?s(r)?n.stylize(RegExp.prototype.toString.call(r),'regexp'):n.stylize('[Object]','special'):(n.seen.push(r),b=j?i(n,r,l,g,c):c.map(function(e){return a(n,r,l,g,e,j)}),n.seen.pop(),u(b,m,z)):z[0]+m+z[1]}function t(n,e){if(g(e))return n.stylize('undefined','undefined');if('string'==typeof e){var r="'"+JSON.stringify(e).replace(/^"|"$/g,'').replace(/'/g,"\\'").replace(/\\"/g,'"')+"'";return n.stylize(r,'string')}return c(e)?n.stylize(''+e,'number'):l(e)?n.stylize(''+e,'boolean'):f(e)?n.stylize('null','null'):void 0}function o(n){return'['+Error.prototype.toString.call(n)+']'}function i(n,e,r,t,o){for(var i=[],u=0,l=e.length;u<l;++u)b(e,String(u))?i.push(a(n,e,r,t,String(u),!0)):i.push('');return o.forEach(function(o){o.match(/^\d+$/)||i.push(a(n,e,r,t,o,!0))}),i}function a(n,e,t,o,i,a){var u,l,c;if((c=Object.getOwnPropertyDescriptor(e,i)||{value:e[i]}).get?l=c.set?n.stylize('[Getter/Setter]','special'):n.stylize('[Getter]','special'):c.set&&(l=n.stylize('[Setter]','special')),b(o,i)||(u='['+i+']'),l||(n.seen.indexOf(c.value)<0?(l=f(t)?r(n,c.value,null):r(n,c.value,t-1)).indexOf('\n')>-1&&(l=a?l.split('\n').map(function(n){return'  '+n}).join('\n').substr(2):'\n'+l.split('\n').map(function(n){return'   '+n}).join('\n')):l=n.stylize('[Circular]','special')),g(u)){if(a&&i.match(/^\d+$/))return l;(u=JSON.stringify(''+i)).match(/^"([a-zA-Z_][a-zA-Z_0-9]*)"$/)?(u=u.substr(1,u.length-2),u=n.stylize(u,'name')):(u=u.replace(/'/g,"\\'").replace(/\\"/g,'"').replace(/(^"|"$)/g,"'"),u=n.stylize(u,'string'))}return u+': '+l}function u(n,e,r){return n.reduce(function(n,e){return 0,e.indexOf('\n')>=0&&0,n+e.replace(/\u001b\[\d\d?m/g,'').length+1},0)>60?r[0]+(''===e?'':e+'\n ')+' '+n.join(',\n  ')+' '+r[1]:r[0]+e+' '+n.join(', ')+' '+r[1]}function l(n){return'boolean'==typeof n}function f(n){return null===n}function c(n){return'number'==typeof n}function g(n){return void 0===n}function s(n){return p(n)&&'[object RegExp]'===v(n)}function p(n){return'object'==typeof n&&null!==n}function y(n){return p(n)&&'[object Date]'===v(n)}function d(n){return p(n)&&('[object Error]'===v(n)||n instanceof Error)}function h(n){return'function'==typeof n}function v(n){return Object.prototype.toString.call(n)}function b(n,e){return Object.prototype.hasOwnProperty.call(n,e)}return function(e,t){return r({seen:[],formatValueCalls:0,stylize:n},e,t.depth)}})(),r='(index)',t={trace:0,info:1,warn:2,error:3},o=[];o[t.trace]='debug',o[t.info]='log',o[t.warn]='warning',o[t.error]='error';var i=1;function a(r){return function(){var a;a=1===arguments.length&&'string'==typeof arguments[0]?arguments[0]:Array.prototype.map.call(arguments,function(n){return e(n,{depth:10})}).join(', ');var u=r;'Warning: '===a.slice(0,9)&&u>=t.error&&(u=t.warn),n.__inspectorLog&&n.__inspectorLog(o[u],a,[].slice.call(arguments),i),g.length&&(a=s('',a)),n.nativeLoggingHook(a,u)}}function u(n,e){return Array.apply(null,Array(e)).map(function(){return n})}var l="\u2502",f="\u2510",c="\u2518",g=[];function s(n,e){return g.join('')+n+' '+(e||'')}if(n.nativeLoggingHook){n.console;n.console={error:a(t.error),info:a(t.info),log:a(t.info),warn:a(t.warn),trace:a(t.trace),debug:a(t.trace),table:function(e){if(!Array.isArray(e)){var o=e;for(var i in e=[],o)if(o.hasOwnProperty(i)){var a=o[i];a[r]=i,e.push(a)}}if(0!==e.length){var l=Object.keys(e[0]).sort(),f=[],c=[];l.forEach(function(n,r){c[r]=n.length;for(var t=0;t<e.length;t++){var o=(e[t][n]||'?').toString();f[t]=f[t]||[],f[t][r]=o,c[r]=Math.max(c[r],o.length)}});for(var g=y(c.map(function(n){return u('-',n).join('')}),'-'),s=[y(l),g],p=0;p<e.length;p++)s.push(y(f[p]));n.nativeLoggingHook('\n'+s.join('\n'),t.info)}else n.nativeLoggingHook('',t.info);function y(n,e){var r=n.map(function(n,e){return n+u(' ',c[e]-n.length).join('')});return e=e||' ',r.join(e+'|'+e)}},group:function(e){n.nativeLoggingHook(s(f,e),t.info),g.push(l)},groupEnd:function(){g.pop(),n.nativeLoggingHook(s(c),t.info)},groupCollapsed:function(e){n.nativeLoggingHook(s(c,e),t.info),g.push(l)},assert:function(e,r){e||n.nativeLoggingHook('Assertion failed: '+r,t.error)}}}else if(!n.console){var p=n.print||function(){};n.console={error:p,info:p,log:p,warn:p,trace:p,debug:p,table:p}}})('undefined'!=typeof globalThis?globalThis:'undefined'!=typeof global?global:'undefined'!=typeof window?window:this);
!(function(n){var r=0,t=function(n,r){throw n},l={setGlobalHandler:function(n){t=n},getGlobalHandler:function(){return t},reportError:function(n){t&&t(n,!1)},reportFatalError:function(n){t&&t(n,!0)},applyWithGuard:function(n,t,u,o,e){try{return r++,n.apply(t,u)}catch(n){l.reportError(n)}finally{r--}return null},applyWithGuardIfNeeded:function(n,r,t){return l.inGuard()?n.apply(r,t):(l.applyWithGuard(n,r,t),null)},inGuard:function(){return!!r},guard:function(n,r,t){var u;if('function'!=typeof n)return console.warn('A function must be passed to ErrorUtils.guard, got ',n),null;var o=null!=(u=null!=r?r:n.name)?u:'<generated guard>';return function(){for(var r=arguments.length,u=new Array(r),e=0;e<r;e++)u[e]=arguments[e];return l.applyWithGuard(n,null!=t?t:this,u,null,o)}}};n.ErrorUtils=l})('undefined'!=typeof globalThis?globalThis:'undefined'!=typeof global?global:'undefined'!=typeof window?window:this);
'undefined'!=typeof globalThis?globalThis:'undefined'!=typeof global?global:'undefined'!=typeof window&&window,(function(){'use strict';var e=Object.prototype.hasOwnProperty;'function'!=typeof Object.entries&&(Object.entries=function(n){if(null==n)throw new TypeError('Object.entries called on non-object');var o=[];for(var t in n)e.call(n,t)&&o.push([t,n[t]]);return o}),'function'!=typeof Object.values&&(Object.values=function(n){if(null==n)throw new TypeError('Object.values called on non-object');var o=[];for(var t in n)e.call(n,t)&&o.push(n[t]);return o})})();
__d(function(g,r,i,a,m,e,d){r(d[0]),r(d[1])},1,[2,55]);
__d(function(g,r,i,a,m,e,d){'use strict';r(d[0]);var t=r(d[1]);m.exports={get AccessibilityInfo(){return r(d[2])},get ActivityIndicator(){return r(d[3])},get ART(){return t('art-moved',"React Native ART has been extracted from react-native core and will be removed in a future release. It can now be installed and imported from '@react-native-community/art' instead of 'react-native'. See https://github.com/react-native-community/art"),r(d[4])},get Button(){return r(d[5])},get CheckBox(){return t('checkBox-moved',"CheckBox has been extracted from react-native core and will be removed in a future release. It can now be installed and imported from '@react-native-community/checkbox' instead of 'react-native'. See https://github.com/react-native-community/react-native-checkbox"),r(d[6])},get DatePickerIOS(){return t('DatePickerIOS-merged',"DatePickerIOS has been merged with DatePickerAndroid and will be removed in a future release. It can now be installed and imported from '@react-native-community/datetimepicker' instead of 'react-native'. See https://github.com/react-native-community/react-native-datetimepicker"),r(d[7])},get DrawerLayoutAndroid(){return r(d[8])},get FlatList(){return r(d[9])},get Image(){return r(d[10])},get ImageBackground(){return r(d[11])},get InputAccessoryView(){return r(d[12])},get KeyboardAvoidingView(){return r(d[13])},get MaskedViewIOS(){return t('maskedviewios-moved',"MaskedViewIOS has been extracted from react-native core and will be removed in a future release. It can now be installed and imported from '@react-native-community/masked-view' instead of 'react-native'. See https://github.com/react-native-community/react-native-masked-view"),r(d[14])},get Modal(){return r(d[15])},get Picker(){return r(d[16])},get PickerIOS(){return r(d[17])},get ProgressBarAndroid(){return r(d[18])},get ProgressViewIOS(){return r(d[19])},get SafeAreaView(){return r(d[20])},get ScrollView(){return r(d[21])},get SectionList(){return r(d[22])},get SegmentedControlIOS(){return r(d[23])},get Slider(){return t('slider-moved',"Slider has been extracted from react-native core and will be removed in a future release. It can now be installed and imported from '@react-native-community/slider' instead of 'react-native'. See https://github.com/react-native-community/react-native-slider"),r(d[24])},get Switch(){return r(d[25])},get RefreshControl(){return r(d[26])},get StatusBar(){return r(d[27])},get Text(){return r(d[28])},get TextInput(){return r(d[29])},get Touchable(){return r(d[30])},get TouchableHighlight(){return r(d[31])},get TouchableNativeFeedback(){return r(d[32])},get TouchableOpacity(){return r(d[33])},get TouchableWithoutFeedback(){return r(d[34])},get View(){return r(d[35])},get VirtualizedList(){return r(d[36])},get VirtualizedSectionList(){return r(d[37])},get ActionSheetIOS(){return r(d[38])},get Alert(){return r(d[39])},get Animated(){return r(d[40])},get AppRegistry(){return r(d[41])},get AppState(){return r(d[42])},get AsyncStorage(){return t('async-storage-moved',"AsyncStorage has been extracted from react-native core and will be removed in a future release. It can now be installed and imported from '@react-native-community/async-storage' instead of 'react-native'. See https://github.com/react-native-community/react-native-async-storage"),r(d[43])},get BackHandler(){return r(d[44])},get Clipboard(){return r(d[45])},get DatePickerAndroid(){return t('DatePickerAndroid-merged',"DatePickerAndroid has been merged with DatePickerIOS and will be removed in a future release. It can now be installed and imported from '@react-native-community/datetimepicker' instead of 'react-native'. See https://github.com/react-native-community/react-native-datetimepicker"),r(d[46])},get DeviceInfo(){return r(d[47])},get Dimensions(){return r(d[48])},get Easing(){return r(d[49])},get findNodeHandle(){return r(d[50]).findNodeHandle},get I18nManager(){return r(d[51])},get ImagePickerIOS(){return t('imagePickerIOS-moved',"ImagePickerIOS has been extracted from react-native core and will be removed in a future release. Please upgrade to use either '@react-native-community/react-native-image-picker' or 'expo-image-picker'. If you cannot upgrade to a different library, please install the deprecated '@react-native-community/image-picker-ios' package. See https://github.com/react-native-community/react-native-image-picker-ios"),r(d[52])},get InteractionManager(){return r(d[53])},get Keyboard(){return r(d[54])},get LayoutAnimation(){return r(d[55])},get Linking(){return r(d[56])},get NativeDialogManagerAndroid(){return r(d[57]).default},get NativeEventEmitter(){return r(d[58])},get PanResponder(){return r(d[59])},get PermissionsAndroid(){return r(d[60])},get PixelRatio(){return r(d[61])},get PushNotificationIOS(){return t('pushNotificationIOS-moved',"PushNotificationIOS has been extracted from react-native core and will be removed in a future release. It can now be installed and imported from '@react-native-community/push-notification-ios' instead of 'react-native'. See https://github.com/react-native-community/react-native-push-notification-ios"),r(d[62])},get Settings(){return r(d[63])},get Share(){return r(d[64])},get StatusBarIOS(){return t('StatusBarIOS-merged','StatusBarIOS has been merged with StatusBar and will be removed in a future release. Use StatusBar for mutating the status bar'),r(d[65])},get StyleSheet(){return r(d[66])},get Systrace(){return r(d[67])},get TimePickerAndroid(){return t('TimePickerAndroid-merged',"TimePickerAndroid has been merged with DatePickerIOS and DatePickerAndroid and will be removed in a future release. It can now be installed and imported from '@react-native-community/datetimepicker' instead of 'react-native'. See https://github.com/react-native-community/react-native-datetimepicker"),r(d[68])},get ToastAndroid(){return r(d[69])},get TurboModuleRegistry(){return r(d[70])},get TVEventHandler(){return r(d[71])},get UIManager(){return r(d[72])},get unstable_batchedUpdates(){return r(d[50]).unstable_batchedUpdates},get useWindowDimensions(){return r(d[73]).default},get UTFSequence(){return r(d[74])},get Vibration(){return r(d[75])},get YellowBox(){return r(d[76])},get DeviceEventEmitter(){return r(d[77])},get NativeAppEventEmitter(){return r(d[78])},get NativeModules(){return r(d[79])},get Platform(){return r(d[80])},get processColor(){return r(d[81])},get requireNativeComponent(){return r(d[82])},get unstable_RootTagContext(){return r(d[83])},get ColorPropType(){return r(d[84])},get EdgeInsetsPropType(){return r(d[85])},get PointPropType(){return r(d[86])},get ViewPropTypes(){return r(d[87])}}},2,[3,4,7,52,183,193,284,287,288,250,271,293,294,296,297,299,306,310,180,311,312,257,279,313,314,316,254,291,194,319,200,326,209,217,210,81,251,280,327,137,218,329,344,347,340,349,351,353,60,246,83,304,354,227,261,263,356,138,121,358,360,59,362,364,365,367,58,28,368,370,12,204,44,372,373,374,376,32,147,13,49,75,178,303,64,196,377,273]);
__d(function(g,r,i,a,m,e,d){'use strict';m.exports=function(n,o,t,f,s,u,c,l){if(!n){var v;if(void 0===o)v=new Error("Minified exception occurred; use the non-minified dev environment for the full error message and additional helpful warnings.");else{var p=[t,f,s,u,c,l],h=0;(v=new Error(o.replace(/%s/g,function(){return p[h++]}))).name='Invariant Violation'}throw v.framesToPop=1,v}}},3,[]);
__d(function(g,r,i,a,m,e,d){'use strict';var t=r(d[0]),n={};m.exports=function(c,o){n[c]||(t(!1,o),n[c]=!0)}},4,[5]);
__d(function(g,r,i,a,m,e,d){'use strict';var t=r(d[0]);m.exports=t},5,[6]);
__d(function(g,r,i,a,m,e,d){"use strict";function t(t){return function(){return t}}var n=function(){};n.thatReturns=t,n.thatReturnsFalse=t(!1),n.thatReturnsTrue=t(!0),n.thatReturnsNull=t(null),n.thatReturnsThis=function(){return this},n.thatReturnsArgument=function(t){return t},m.exports=n},6,[]);
__d(function(g,r,i,a,m,e,d){'use strict';var n=r(d[0])(r(d[1])),t=r(d[2]),o=r(d[3]),s=new Map,c={isBoldTextEnabled:function(){return Promise.resolve(!1)},isGrayscaleEnabled:function(){return Promise.resolve(!1)},isInvertColorsEnabled:function(){return Promise.resolve(!1)},isReduceMotionEnabled:function(){return new Promise(function(t,o){n.default?n.default.isReduceMotionEnabled(t):o(!1)})},isReduceTransparencyEnabled:function(){return Promise.resolve(!1)},isScreenReaderEnabled:function(){return new Promise(function(t,o){n.default?n.default.isTouchExplorationEnabled(t):o(!1)})},get fetch(){return this.isScreenReaderEnabled},addEventListener:function(n,o){var c;'change'===n||'screenReaderChanged'===n?c=t.addListener("touchExplorationDidChange",function(n){o(n)}):'reduceMotionChanged'===n&&(c=t.addListener("reduceMotionDidChange",function(n){o(n)})),s.set(o,c)},removeEventListener:function(n,t){var o=s.get(t);o&&(o.remove(),s.delete(t))},setAccessibilityFocus:function(n){o.sendAccessibilityEvent(n,o.getConstants().AccessibilityEventTypes.typeViewFocused)},announceForAccessibility:function(t){n.default&&n.default.announceForAccessibility(t)}};m.exports=c},7,[8,9,32,44]);
__d(function(g,r,i,a,m,e,d){m.exports=function(n){return n&&n.__esModule?n:{default:n}}},8,[]);
__d(function(g,r,i,a,m,e,d){'use strict';var t=r(d[0]);Object.defineProperty(e,"__esModule",{value:!0}),e.default=void 0;var u=t(r(d[1])).get('AccessibilityInfo');e.default=u},9,[10,12]);
__d(function(g,r,i,a,m,e,d){var t=r(d[0]);function n(){if("function"!=typeof WeakMap)return null;var t=new WeakMap;return n=function(){return t},t}m.exports=function(o){if(o&&o.__esModule)return o;if(null===o||"object"!==t(o)&&"function"!=typeof o)return{default:o};var u=n();if(u&&u.has(o))return u.get(o);var f={},c=Object.defineProperty&&Object.getOwnPropertyDescriptor;for(var p in o)if(Object.prototype.hasOwnProperty.call(o,p)){var l=c?Object.getOwnPropertyDescriptor(o,p):null;l&&(l.get||l.set)?Object.defineProperty(f,p,l):f[p]=o[p]}return f.default=o,u&&u.set(o,f),f}},10,[11]);
__d(function(g,r,i,a,m,e,d){function o(t){"@babel/helpers - typeof";return"function"==typeof Symbol&&"symbol"==typeof("function"==typeof Symbol?Symbol.iterator:"@@iterator")?m.exports=o=function(o){return typeof o}:m.exports=o=function(o){return o&&"function"==typeof Symbol&&o.constructor===Symbol&&o!==("function"==typeof Symbol?Symbol.prototype:"@@prototype")?"symbol":typeof o},o(t)}m.exports=o},11,[]);
...
...
__r(86);
__r(1);
```

### (2) Metro工具
随着React Native 版本迭代，官方已经逐步将bundle文件生成流程规范化，并为此设计了独立的打包模块 – Metro。Metro 通过输入一个需要打包的JS文件及几个配置参数，返回一个包含了所有依赖内容的JS文件。
Metro将打包的过程分为了3个依次执行的阶段：

**解析（Resolution）**：计算得到所有的依赖模块，形成依赖树，该过程是多线程并行执行。

**转义（Transformation）**：将模块内容转义为React Native可识别的格式，该过程是多线程并行执行。

**序列化（Serialization）**：将所有的模块合并到一个文件中输出。
Metro工具提供了配置功能，开发人员可以通过配置RN项目中的metro.config.js文件修改bundle文件的生成流程。

新拆包方法主要关注的是Metro工具在“序列化”阶段时调用的 `createModuleIdFactory(path)`方法和`processModuleFilter(module)`。
`createModuleIdFactory(path)`是传入的模块绝对路径path，并为该模块返回一个唯一的Id。`processModuleFilter(module)`则可以实现对模块进行过滤，使其不被写入到最后的bundle文件中。

官方的`createModuleIdFactory(path)`方法是返回个数字。(如前所述，该数字在 require 方法中进行被调用，以此来实现模块的导入和初始化)
```js
"use strict";
function createModuleIdFactory() {
  const fileToIdMap = new Map();
  let nextId = 0;
  return path => {
    let id = fileToIdMap.get(path);
    if (typeof id !== "number") {
      id = nextId++;
      fileToIdMap.set(path, id);
    }
    return id;
  };
}
```
官方的实现存在的问题是Id值从0开始分配，所以**任意改动业务代码可能引起模块构建的顺序变动**，致使同一个模块在两次构建分配了有2个不同的Id值。

针对官方实现的问题，我们重新声明一个`createModuleIdFactory(path)`方法，该方法使用**当前模块文件的路径的哈希值**作为分配模块的Id的依据，并建立哈希值与模块Id对应关系的存在**本地文件module_id.json**中，每次编译Bundle文件前先读取本地关系文件来初始化内部缓存，当需要分配Id时，先从内部缓存中查找，查找不到则新分配Id并存储变化。

由上述步骤可以到达同一个模块，无论编译顺序如何，返回的Id是同一个。关键代码如下：
```js
/**
 * Get the key, which used to find the Id in local storage
 * @param {get} path
 */
function getFindKey(path) {
  let md5 = crypto.createHash('md5');
  md5.update(path);
  let findKey = md5.digest('hex');
  return findKey;
}

var moduleIdsJsonObj = {};

const moduleIdsMapFilePath = './module_id.json';

/**
 * 注释: 创建模块ID
 * 时间: 2020/6/12 0012 15:40
 * @author 郭翰林
 */
buildCreateModuleIdFactoryWithLocalStorage = function(buildConfig) {
  let currentModuleId = 0;
  // init moduleIdsJsonObj from file;
  moduleIdsJsonObj = getOrCreateModuleIdsJsonObj(moduleIdsMapFilePath);
  // init currentModuleId;
  for (var key in moduleIdsJsonObj) {
    currentModuleId = currentModuleId > moduleIdsJsonObj[key].id ? currentModuleId : moduleIdsJsonObj[key].id;
  }
  console.log('currentModuleId = ' + currentModuleId);
  return () => {
    return path => {
      // console.log(`buildType: ${buildType}`);
      let findKey = getFindKey(path);
      if (moduleIdsJsonObj[findKey] == null) {
        moduleIdsJsonObj[findKey] = {
          id: ++currentModuleId,
          type: buildConfig.type,
        };
        saveModuleIdsJsonObj(moduleIdsMapFilePath, moduleIdsJsonObj);
      }
      let id = moduleIdsJsonObj[findKey].id;
      // console.log(`createModuleIdFactory id = ${id} for ${path}`);
      return id;
    };
  };
};

```
同时，为了能够在`processModuleFilter(module)`方法中对模块进行过滤，需要在构建Common文件时，标记某个模块是否已包含在Common文件中。为此，我们在保存模块id对应关系时，额外加上了type字段，该字段的值来源于构建脚本执行时传入的参数。当构建Common文件时，该值为common，当构建Diff文件时，该值为diff。

生成的`module_id.json`文件如下：
```json
{
    "f8f41b41b631cbda0ab96da29ca046e8": {
        "id": 1,
        "type": "common"
    },
    "6c5b9eb9940d40c69dee43d88f1beb4c": {
        "id": 2,
        "type": "common"
    },
    "dd51f8177f72708a90b7826e9e370fc9": {
        "id": 3,
        "type": "common"
    },
   ...
   ...
    "95e6c1577450804611cbd475e01dc540": {
        "id": 1191,
        "type": "common"
    },
    "d5339447c8123c7c23ce432f3b3c3671": {
        "id": 1192,
        "type": "common"
    }
}
```
`processModuleFilter(module)`方法实现如下：
```js
/**
 * 注释: 过滤模块ID
 * 时间: 2020/6/12 0012 15:41
 * @author 郭翰林
 * @param buildConfig
 * @returns {function(...[*]=)}
 */
buildProcessModuleFilter = function(buildConfig) {
  return moduleObj => {
    let path = moduleObj.path;
    if (!fs.existsSync(path)) {
      return true;
    }
    if (buildConfig.type === BUILD_TYPE_DIFF) {
      //当前打包是否是diff打包
      let findKey = getFindKey(path);
      let storeObj = moduleIdsJsonObj[findKey];
      if (storeObj != null && storeObj.type === BUILD_TYPE_COMMON) {
        //如果diff包中存在的模块，common中已经存在则过滤掉，不打进最后的bundle中
        return false;
      }
      return true;
    }
    return true;
  };
};
```
通过上述步骤构建出的Diff文件中，还保留了Pollyfills部分内容，需要进行删除。删除脚步位于removePolyfill.js中，代码如下：
```js
const fs = require('fs');
const readline = require('readline');
// Functions

// Main
let argvs = process.argv.splice(2);
let filePath = argvs[0];

var fRead = fs.createReadStream(filePath);
var objReadline = readline.createInterface({
  input: fRead,
});
let diff = new Array();
objReadline.on('line', function(line) {
  if (line.startsWith('__d') || line.startsWith('__r')) {
    diff.push(line);
  }
});
objReadline.on('close', function() {
  let data = diff.join('\n');
  fs.writeFileSync(filePath, data);
});

//删除module_id.json
fs.access('module_id.json', fs.constants.F_OK, error => {
  if (!error) {
    fs.unlink('module_id.json', err => {
      if (err) {
        console.log(err);
      }
    });
  }
});

```

## 二、建立分包脚本进行分包操作
Android中React Native打包主要是通过react.gradle文件去执行官方打包命令，我们可以到react.gradle中查看打包关键代码：

```
if (bundleConfig) {
    extraArgs = extraArgs.clone()
    extraArgs.add("--config");
    extraArgs.add(bundleConfig);
}

if (Os.isFamily(Os.FAMILY_WINDOWS)) {
    commandLine("cmd", "/c", *nodeExecutableAndArgs, cliPath, bundleCommand, "--platform", "android", "--dev", "${devEnabled}",
        "--reset-cache", "--entry-file", entryFile, "--bundle-output", jsBundleFile, "--assets-dest", resourcesDir,
        "--sourcemap-output", enableHermes ? jsPackagerSourceMapFile : jsOutputSourceMapFile, *extraArgs)
} else {
    commandLine(*nodeExecutableAndArgs, cliPath, bundleCommand, "--platform", "android", "--dev", "${devEnabled}",
        "--reset-cache", "--entry-file", entryFile, "--bundle-output", jsBundleFile, "--assets-dest", resourcesDir,
        "--sourcemap-output", enableHermes ? jsPackagerSourceMapFile : jsOutputSourceMapFile, *extraArgs)
}

```
打包代码中有一个`--config`参数，这边就是配置我们之前书写的Metro打包脚本，我把Metro打包脚本分为基础包脚本、diff包脚本、公用打包脚本，代码如下：

公用打包脚本  `metro.config.base.js`：

```js
/**
 * Metro configuration for React Native
 * https://github.com/facebook/react-native
 *
 * @format
 */
const fs = require('fs');
const crypto = require('crypto');

const BUILD_TYPE_COMMON = 'common';
const BUILD_TYPE_DEFAULT = 'default';
const BUILD_TYPE_DIFF = 'diff';

const moduleIdsMapFilePath = './module_id.json';

/**
 *
 * @param {*} filepath
 */
function getOrCreateModuleIdsJsonObj(filepath) {
  if (fs.existsSync(filepath)) {
    console.log(`init map from file : ${filepath}`);
    let data = fs.readFileSync(filepath, 'utf-8');
    return JSON.parse(data);
  } else {
    return {};
  }
}
/**
 *
 * @param {*} filepath
 * @param {*} jsonObj
 */
function saveModuleIdsJsonObj(filepath, jsonObj) {
  let data = JSON.stringify(jsonObj);
  fs.writeFileSync(filepath, data, err => {
    if (err) throw err;
    console.log(`Save ${filepath} SUCCESS.`);
  });
}
/**
 * Get the key, which used to find the Id in local storage
 * @param {get} path
 */
function getFindKey(path) {
  let md5 = crypto.createHash('md5');
  md5.update(path);
  let findKey = md5.digest('hex');
  return findKey;
}

var moduleIdsJsonObj = {};

/**
 * 注释: 创建模块ID
 * 时间: 2020/6/12 0012 15:40
 * @author 郭翰林
 */
buildCreateModuleIdFactoryWithLocalStorage = function(buildConfig) {
  let currentModuleId = 0;
  // init moduleIdsJsonObj from file;
  moduleIdsJsonObj = getOrCreateModuleIdsJsonObj(moduleIdsMapFilePath);
  // init currentModuleId;
  for (var key in moduleIdsJsonObj) {
    currentModuleId = currentModuleId > moduleIdsJsonObj[key].id ? currentModuleId : moduleIdsJsonObj[key].id;
  }
  console.log('currentModuleId = ' + currentModuleId);
  return () => {
    return path => {
      // console.log(`buildType: ${buildType}`);
      let findKey = getFindKey(path);
      if (moduleIdsJsonObj[findKey] == null) {
        moduleIdsJsonObj[findKey] = {
          id: ++currentModuleId,
          type: buildConfig.type,
        };
        saveModuleIdsJsonObj(moduleIdsMapFilePath, moduleIdsJsonObj);
      }
      let id = moduleIdsJsonObj[findKey].id;
      // console.log(`createModuleIdFactory id = ${id} for ${path}`);
      return id;
    };
  };
};

/**
 * 注释: 过滤模块ID
 * 时间: 2020/6/12 0012 15:41
 * @author 郭翰林
 * @param buildConfig
 * @returns {function(...[*]=)}
 */
buildProcessModuleFilter = function(buildConfig) {
  return moduleObj => {
    let path = moduleObj.path;
    if (!fs.existsSync(path)) {
      return true;
    }
    if (buildConfig.type === BUILD_TYPE_DIFF) {
      let findKey = getFindKey(path);
      let storeObj = moduleIdsJsonObj[findKey];
      if (storeObj != null && storeObj.type === BUILD_TYPE_COMMON) {
        return false;
      }
      return true;
    }
    return true;
  };
};

module.exports = {
  BuildType: {
    COMMON: BUILD_TYPE_COMMON,
    DEFAULT: BUILD_TYPE_DEFAULT,
    DIFF: BUILD_TYPE_DIFF,
  },
  buildCreateModuleIdFactory: buildCreateModuleIdFactoryWithLocalStorage,
  buildProcessModuleFilter: buildProcessModuleFilter,
};

```
基础包打包脚本 `metro.config.common.js`:
```js
/**
 * Metro configuration for React Native
 * https://github.com/facebook/react-native
 *
 * @format
 */

const baseMetroConfig = require('./metro.config.base.js');
const buildConfig = {
  type: baseMetroConfig.BuildType.COMMON,
};

module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: false,
      },
    }),
  },
  serializer: {
    createModuleIdFactory: baseMetroConfig.buildCreateModuleIdFactory(buildConfig),
    processModuleFilter: baseMetroConfig.buildProcessModuleFilter(buildConfig),
  },
};

```
diff包打包脚本 `metro.config.diff.js`:
```js
/**
 * Metro configuration for React Native
 * https://github.com/facebook/react-native
 *
 * @format
 */

const baseMetroConfig = require('./metro.config.base.js');
const buildConfig = {
  type: baseMetroConfig.BuildType.DIFF,
};
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: false,
      },
    }),
  },
  serializer: {
    createModuleIdFactory: baseMetroConfig.buildCreateModuleIdFactory(buildConfig),
    processModuleFilter: baseMetroConfig.buildProcessModuleFilter(buildConfig),
  },
};
```

下面我们就可以在工程`package.json`中配置基础包和diff包打包命令：
```
  "scripts": {
    .....
    "prettier": "prettier --write app/**/*.js app/**/*.jsx app/**/*.ts app/**/*.tsx",
    "build_android_common_bundle": "node scripts/bundle/createAndroidAssets.js && react-native bundle --platform android --dev false --entry-file app/entry/Common.js  --bundle-output ./android/app/build/generated/assets/react/release/index.android.bundle --assets-dest ./android/app/build/generated/res/react/release/ --sourcemap-output ./android/app/build/generated/sourcemaps/react/release/index.android.bundle.map --config metro.config.common.js",
    "build_android_diff_bundle": "react-native bundle --platform android --dev false --entry-file index.android.js  --bundle-output ./android/app/build/generated/assets/react/release/diff.android.bundle --assets-dest ./android/app/build/generated/res/react/release/ --sourcemap-output ./android/app/build/generated/sourcemaps/react/release/diff.android.bundle.map --config metro.config.diff.js && node scripts/bundle/removePolyfill.js ./android/app/build/generated/assets/react/release/diff.android.bundle"
  },
```
这里注意到在生成基础包之前，我执行了一次node命令，这个命令是用来**生成bundle输出文件夹和最终的sourcemaps文件夹**的，要不然执行打包脚本会报错`createAndroidAssets.js`脚本如下:
```js
const fs = require('fs');
const path = require('path');
//删除module_id.json
fs.access('module_id.json', fs.constants.F_OK, error => {
  if (!error) {
    fs.unlink('module_id.json', err => {
      if (err) {
        console.log(err);
      }
    });
  }
});
//Android app模块下创建相应文件夹
fs.access(path.join(__dirname, '../../android/app/'), fs.constants.F_OK, error => {
  if (!error) {
    fs.mkdir(
      path.join(__dirname, '../../android/app/build/generated/assets/react/release'),
      {recursive: true},
      error => {
        if (error) {
          console.log(error);
        }
      },
    );
  }
});
fs.access(path.join(__dirname, '../../android/app/'), fs.constants.F_OK, error => {
  if (!error) {
    fs.mkdir(path.join(__dirname, '../../android/app/build/generated/res/react/release'), {recursive: true}, error => {
      if (error) {
        console.log(error);
      }
    });
  }
});
fs.access(path.join(__dirname, '../../android/app/'), fs.constants.F_OK, error => {
  if (!error) {
    fs.mkdir(
      path.join(__dirname, '../../android/app/build/generated/sourcemaps/react/release/'),
      {recursive: true},
      error => {
        if (error) {
          console.log(error);
        }
      },
    );
  }
});

```
此时!我们已经可以通过下面的2条命令去生成基础包和diff包了：
```js
npm run build_android_common_bundle
npm run build_android_diff_bundle
```
## 三、基础包和diff包的划分，以及Android中bundle的异步加载
### （1）基础包、diff包划分
以我们工程为例，我把App引导页和主页作为基础模块，放到Common.js中，代码如下，下面的代码我们无需过多的关心，我们只需关心基础包会把哪些内容打进包内：
```js
/**
 * 注释: RN 基础模块
 * 时间: 2020/6/15 0015 10:05
 * @author 郭翰林
 */
import React, {PureComponent} from 'react';
import {
  ActivityIndicator,
  AppRegistry,
  DeviceEventEmitter,
  FlatList,
  Image,
  NativeModules,
  Platform,
  ScrollView,
  StatusBar,
  Text,
  TextInput,
  View,
  YellowBox,
} from 'react-native';
import KeyboardManager from 'react-native-keyboard-manager';
import {CommonBridge, PageBridge} from '../bridge';
import moment from 'moment-timezone';
import {Provider} from '@ant-design/react-native';
import {enableScreens} from 'react-native-screens';
import AsyncStorage from '@react-native-community/async-storage';
import Sentry from '../dependence/sentry';
import Config from '../config';
import {createStackNavigator} from 'react-navigation-stack';
import {injectEventLog} from '../commons/event';
import {ComponentStyles, StyleConfig} from '../resources/style';
import Button from 'react-native-button';
import {TransitionIOSSpec} from 'react-navigation-stack/src/vendor/TransitionConfigs/TransitionSpecs';
import {forHorizontalIOS} from 'react-navigation-stack/src/vendor/TransitionConfigs/CardStyleInterpolators';
import {createAppContainer, NavigationActions, StackActions} from 'react-navigation';
import IntroducePage from '../modules/introduce/IntroducePage';
import MainTabBar from '../modules/main/MainTabBar';
import ProductSearchPage from '../modules/product/list/ProductSearchPage';
import AnXin from '../modules/anXin/router';

moment.tz.setDefault('Asia/Shanghai');

Platform.OS !== 'ios' && enableScreens();

let shouldLoadNaviState = true;

/**
 * 注释: 以欢迎页、首页以及相关依赖页面作为基础包
 * 时间: 2020/6/15 0015 10:23
 * @author 郭翰林
 * @returns {{new(*=): {onNavigationStateChange: function(*, *=): void, render: {(): *, (): React.ReactNode}, componentDidMount?(): void, shouldComponentUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): boolean, componentWillUnmount?(): void, componentDidCatch?(error: Error, errorInfo: React.ErrorInfo): void, getSnapshotBeforeUpdate?(prevProps: Readonly<{}>, prevState: Readonly<{}>): (any | null), componentDidUpdate?(prevProps: Readonly<{}>, prevState: Readonly<{}>, snapshot?: any): void, componentWillMount?(): void, UNSAFE_componentWillMount?(): void, componentWillReceiveProps?(nextProps: Readonly<{}>, nextContext: any): void, UNSAFE_componentWillReceiveProps?(nextProps: Readonly<{}>, nextContext: any): void, componentWillUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): void, UNSAFE_componentWillUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): void, context: any, setState<K extends never>(state: (((prevState: Readonly<{}>, props: Readonly<{}>) => (Pick<{}, K> | {} | null)) | Pick<{}, K> | {} | null), callback?: () => void): void, forceUpdate(callback?: () => void): void, readonly props: Readonly<{}> & Readonly<{children?: React.ReactNode}>, state: Readonly<{}>, refs: {[p: string]: React.ReactInstance}}, contextType?: React.Context<any>, new<P, S>(props: Readonly<{}>): {onNavigationStateChange: function(*, *=): void, render: {(): *, (): React.ReactNode}, componentDidMount?(): void, shouldComponentUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): boolean, componentWillUnmount?(): void, componentDidCatch?(error: Error, errorInfo: React.ErrorInfo): void, getSnapshotBeforeUpdate?(prevProps: Readonly<{}>, prevState: Readonly<{}>): (any | null), componentDidUpdate?(prevProps: Readonly<{}>, prevState: Readonly<{}>, snapshot?: any): void, componentWillMount?(): void, UNSAFE_componentWillMount?(): void, componentWillReceiveProps?(nextProps: Readonly<{}>, nextContext: any): void, UNSAFE_componentWillReceiveProps?(nextProps: Readonly<{}>, nextContext: any): void, componentWillUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): void, UNSAFE_componentWillUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): void, context: any, setState<K extends never>(state: (((prevState: Readonly<{}>, props: Readonly<{}>) => (Pick<{}, K> | {} | null)) | Pick<{}, K> | {} | null), callback?: () => void): void, forceUpdate(callback?: () => void): void, readonly props: Readonly<{}> & Readonly<{children?: React.ReactNode}>, state: Readonly<{}>, refs: {[p: string]: React.ReactInstance}}, new<P, S>(props: {}, context?: any): {onNavigationStateChange: function(*, *=): void, render: {(): *, (): React.ReactNode}, componentDidMount?(): void, shouldComponentUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): boolean, componentWillUnmount?(): void, componentDidCatch?(error: Error, errorInfo: React.ErrorInfo): void, getSnapshotBeforeUpdate?(prevProps: Readonly<{}>, prevState: Readonly<{}>): (any | null), componentDidUpdate?(prevProps: Readonly<{}>, prevState: Readonly<{}>, snapshot?: any): void, componentWillMount?(): void, UNSAFE_componentWillMount?(): void, componentWillReceiveProps?(nextProps: Readonly<{}>, nextContext: any): void, UNSAFE_componentWillReceiveProps?(nextProps: Readonly<{}>, nextContext: any): void, componentWillUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): void, UNSAFE_componentWillUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): void, context: any, setState<K extends never>(state: (((prevState: Readonly<{}>, props: Readonly<{}>) => (Pick<{}, K> | {} | null)) | Pick<{}, K> | {} | null), callback?: () => void): void, forceUpdate(callback?: () => void): void, readonly props: Readonly<{}> & Readonly<{children?: React.ReactNode}>, state: Readonly<{}>, refs: {[p: string]: React.ReactInstance}}, prototype: {onNavigationStateChange: function(*, *=): void, render: {(): *, (): React.ReactNode}, componentDidMount?(): void, shouldComponentUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): boolean, componentWillUnmount?(): void, componentDidCatch?(error: Error, errorInfo: React.ErrorInfo): void, getSnapshotBeforeUpdate?(prevProps: Readonly<{}>, prevState: Readonly<{}>): (any | null), componentDidUpdate?(prevProps: Readonly<{}>, prevState: Readonly<{}>, snapshot?: any): void, componentWillMount?(): void, UNSAFE_componentWillMount?(): void, componentWillReceiveProps?(nextProps: Readonly<{}>, nextContext: any): void, UNSAFE_componentWillReceiveProps?(nextProps: Readonly<{}>, nextContext: any): void, componentWillUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): void, UNSAFE_componentWillUpdate?(nextProps: Readonly<{}>, nextState: Readonly<{}>, nextContext: any): void, context: any, setState<K extends never>(state: (((prevState: Readonly<{}>, props: Readonly<{}>) => (Pick<{}, K> | {} | null)) | Pick<{}, K> | {} | null), callback?: () => void): void, forceUpdate(callback?: () => void): void, readonly props: Readonly<{}> & Readonly<{children?: React.ReactNode}>, state: Readonly<{}>, refs: {[p: string]: React.ReactInstance}}}}
 */
function createEntry(pageName) {
  return class extends PureComponent {
    onNavigationStateChange = (prevState, currentState) => {
      processSlideGesture(currentState, this.props.pageId);
    };

    constructor(props) {
      super(props);
      this.state = {
        barStyle: 'dark-content',
      };
      if (pageName === 'MainTabBar' && !this.props.lastAppVersion) {
        this.Router = initialRoute('IntroducePage');
      } else {
        this.Router = initialRoute(pageName);
      }
      if (Platform.OS === 'ios') {
        KeyboardManager.setToolbarPreviousNextButtonEnable(true);
      }
      if (__DEV__) {
        YellowBox.ignoreWarnings([
          'Remote debugger',
          'Warning: isMounted(...) is deprecated',
          'Module RCTImageLoader',
          'You should only render one navigator explicitly',
        ]);
        console.ignoredYellowBox = ['Remote debugger'];
      }
    }

    render() {
      const Router = this.Router;
      return (
        <View style={{flex: 1}}>
          <Provider>
            {Platform.OS === 'ios' ? <StatusBar barStyle={this.state.barStyle} /> : null}
            <Router
              screenProps={this.props}
              {...getPersistenceFunctions()}
              renderLoadingExperimental={() => <ActivityIndicator />}
              onNavigationStateChange={this.onNavigationStateChange.bind(this)}
            />
          </Provider>
        </View>
      );
    }
  };
}

/**
 * 尽量以路由的跳转方式跳转，以减少这边的页面依赖关系
 * @type {{IntroducePage: {screen: IntroducePage}, ProductSearchPage: {screen: ProductSearchPage}, AddFamilyMemberPage: {screen}, MainTabBar: {screen: import("react-navigation").NavigationNavigator<any, import("react-navigation").NavigationProp<import("react-navigation").NavigationState>>}}}
 */
const allPages = {
  IntroducePage: {screen: IntroducePage},
  MainTabBar: {screen: MainTabBar},
  ProductSearchPage: {screen: ProductSearchPage},
  ...AnXin,
};

let initialRoute;
export default (initialRoute = rootName => {
  const navigator = createStackNavigator(injectEventLog(allPages, rootName), {
    initialRouteName: rootName,
    defaultNavigationOptions: props => ({
      headerBackTitle: null,
      headerStyle: ComponentStyles.navigationBar,
      headerLeft: () => (
        <Button
          containerStyle={{marginLeft: 16, width: 24, height: 24}}
          onPress={() => {
            const {params = {}} = props.navigation.state;
            if (params.hud) {
              params.hud.dismiss();
            }
            if (params.onBack) {
              params.onBack();
            } else {
              if (!props.navigation.isFirstRouteInParent()) {
                props.navigation.goBack();
              } else {
                NativeModules.CommonUtilBridge.pop(props.screenProps.pageId);
              }
            }
          }}>
          <Image
            style={{tintColor: '#1D2023', width: 24, height: 24}}
            source={require('../resources/images/common/icon_back.png')}
          />
        </Button>
      ),
      headerRight: () => <View />,
      headerTitleStyle: ComponentStyles.headerTitleStyle,
      headerTitleAlign: 'left',
      headerTintColor: StyleConfig.color_button_normal,
      cardStyle: {backgroundColor: '#fff'},
      headerTitleAllowFontScaling: false,
      cardOverlayEnabled: true,
      cardShadowEnabled: true,
      gestureEnabled: true,
      gestureResponseDistance: {
        horizontal: 10,
      },
      gestureDirection: 'horizontal',
      transitionSpec: {
        open: TransitionIOSSpec,
        close: TransitionIOSSpec,
      },
      cardStyleInterpolator: forHorizontalIOS,
    }),
    headerMode: 'screen',
    mode: 'card',
  });
  navigator.router.getStateForAction = navigateOnce(navigator.router.getStateForAction);
  return createAppContainer(navigator);
});

const navigateOnce = getStateForAction => (action, state) => {
  const {type, routeName, params = {}} = action;
  return state &&
    (type === NavigationActions.NAVIGATE || type === StackActions.PUSH) &&
    !params.canPush &&
    routeName === state.routes[state.routes.length - 1].routeName
    ? null
    : getStateForAction(action, state);
  // you might want to replace 'null' with 'state' if you're using redux (see comments below)
};

AppRegistry.registerComponent('MainEntry', () => createEntry('MainTabBar'));

```
这里的打包会把`Common.js`所有相关的引用以及`React-Navigation`导航配置的路由页面都会被打进包内！
此时，我们再去执行diff包命令，则会把App RN部分**非这部分包内的引用和相关页面打进diff包内**！
### （2）Android中异步加载Bundle
由于我们工程使用的是`React-Navigation`去跳转页面，如果其他的页面在diff包内，则如果在没有加载diff包的时候，是**无法通过`this.props.navigation.push("XXXX")`进行跳转的**，此时就需要我们去改造基础包中页面的页面跳转方式，改为使用**原生路由的方式**的去跳转页面,跳转时新建一个RN容器。另外使用路由的跳转方式也可以**减少基础包中页面的依赖**，使得基础变得更小！
```js
this.props.navigation.navigate('ProductDetailPage', {planId: item.planId});
//改为
RouterPageBridge.gotoRouterSkipSystem(RouterUri.ProductDetailPage, {planId: item.planId});
```

在**基础包加载完成之后设置监听立即去加载diff包**，这样做的好处是可以**减少首页的加载时间**更快的进入首页（但是由于React Native在0.56版本之后，把RN JS代码和资源文件分开之后，这部分时间提升有限！因为大头都是资源文件，而资源文件都是直接被打进APK包assets文件内的）。另一个好处就是首页进行跳转diff包页面时不会出现白屏现象，因为diff包已经被异步加载完成！关键代码如下：
```java
//RN初始化入口页
@Override
public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    ......
    ......
    LoadAnimationView.show(this);
    if(!BuildConfig.DEBUG){
        //设置RN加载监听
        getReactInstanceManager().addReactInstanceEventListener(context -> {
            //异步加载Diff模块
            getReactInstanceManager().getCurrentReactContext().getCatalystInstance().loadScriptFromAssets(getAssets(), "assets://diff.android.bundle", false);
        });
    }
}
```

## 四、Sentry编译问题解决

我们工程改写了react.gradle脚本，在打Release包时，去除原有的打包脚本，从而执行前面我们自己定义的npm 分包脚本，关键代码如下：

```
// Copyright (c) Facebook, Inc. and its affiliates.

// This source code is licensed under the MIT license found in the
// LICENSE file in the root directory of this source tree.

import org.apache.tools.ant.taskdefs.condition.Os

def config = project.hasProperty("react") ? project.react : [];

def cliPath = config.cliPath ?: "node_modules/react-native/cli.js"
def composeSourceMapsPath = config.composeSourceMapsPath ?: "node_modules/react-native/scripts/compose-source-maps.js"
def bundleAssetName = config.bundleAssetName ?: "index.android.bundle"
def entryFile = config.entryFile ?: "index.android.js"
def bundleCommand = config.bundleCommand ?: "bundle"
def reactRoot = file(config.root ?: "../../")
def inputExcludes = config.inputExcludes ?: ["android/**", "ios/**"]
def bundleConfig = config.bundleConfig ? "${reactRoot}/${config.bundleConfig}" : null;
def enableVmCleanup = config.enableVmCleanup == null ? true : config.enableVmCleanup
def hermesCommand = config.hermesCommand ?: "../../node_modules/hermes-engine/%OS-BIN%/hermes"

def reactNativeDevServerPort() {
    def value = project.getProperties().get("reactNativeDevServerPort")
    return value != null ? value : "8081"
}

def reactNativeInspectorProxyPort() {
    def value = project.getProperties().get("reactNativeInspectorProxyPort")
    return value != null ? value : reactNativeDevServerPort()
}

/**
 * 注释：是否开启编译JsBundle任务
 * 时间：2019/4/18 0018 10:18
 * 作者：郭翰林
 * @return
 */
boolean isEnableBuildJsBundle(String targetName) {
    if (targetName.toLowerCase().contains("release")) {
        return false
    }
    File jsBundle = file("$buildDir/intermediates/assets/debug/index.android.bundle")
    if (!jsBundle.exists()) {
        return true
    } else {
        println("【跳过编译JsBundle】JsBundle已存在,无需再次编译")
        return false
    }
}

def getHermesOSBin() {
    if (Os.isFamily(Os.FAMILY_WINDOWS)) return "win64-bin";
    if (Os.isFamily(Os.FAMILY_MAC)) return "osx-bin";
    if (Os.isOs(null, "linux", "amd64", null)) return "linux64-bin";
    throw new Exception("OS not recognized. Please set project.ext.react.hermesCommand " +
            "to the path of a working Hermes compiler.");
}

// Make sure not to inspect the Hermes config unless we need it,
// to avoid breaking any JSC-only setups.
def getHermesCommand = {
    // If the project specifies a Hermes command, don't second guess it.
    if (!hermesCommand.contains("%OS-BIN%")) {
        return hermesCommand
    }

    // Execution on Windows fails with / as separator
    return hermesCommand
            .replaceAll("%OS-BIN%", getHermesOSBin())
            .replace('/' as char, File.separatorChar);
}

// Set enableHermesForVariant to a function to configure per variant,
// or set `enableHermes` to True/False to set all of them
def enableHermesForVariant = config.enableHermesForVariant ?: {
    def variant -> config.enableHermes ?: false
}

android {
    buildTypes.all {
        resValue "integer", "react_native_dev_server_port", reactNativeDevServerPort()
        resValue "integer", "react_native_inspector_proxy_port", reactNativeInspectorProxyPort()
    }
}

afterEvaluate {
    def isAndroidLibrary = plugins.hasPlugin("com.android.library")
    def variants = isAndroidLibrary ? android.libraryVariants : android.applicationVariants
    variants.all { def variant ->
        ....
        ....
        ....
        
        def enableHermes = enableHermesForVariant(variant)
        def currentBundleTask = tasks.create(
                name: "bundle${targetName}JsAndAssets",
                type: Exec) {
            group = "react"
            description = "bundle JS and assets for ${targetName}."

            // Create dirs if they are not there (e.g. the "clean" task just ran)
            doFirst {
                jsBundleDir.mkdirs()
                resourcesDir.mkdirs()
                jsIntermediateSourceMapsDir.mkdirs()
                jsSourceMapsDir.mkdirs()
            }

            .....

            if (isEnableBuildJsBundle(targetName)) {
                if (Os.isFamily(Os.FAMILY_WINDOWS)) {
                    commandLine("cmd", "/c", *nodeExecutableAndArgs, cliPath, bundleCommand, "--platform", "android", "--dev", "${devEnabled}",
                            "--reset-cache", "--entry-file", entryFile, "--bundle-output", jsBundleFile, "--assets-dest", resourcesDir,
                            "--sourcemap-output", enableHermes ? jsPackagerSourceMapFile : jsOutputSourceMapFile, *extraArgs)
                } else {
                    commandLine(*nodeExecutableAndArgs, cliPath, bundleCommand, "--platform", "android", "--dev", "${devEnabled}",
                            "--reset-cache", "--entry-file", entryFile, "--bundle-output", jsBundleFile, "--assets-dest", resourcesDir,
                            "--sourcemap-output", enableHermes ? jsPackagerSourceMapFile : jsOutputSourceMapFile, *extraArgs)
                }

                if (enableHermes) {
                   ......
                   ......
                }
            } else if (targetName.toLowerCase().contains("release")) {
                //Release打包执行分包脚本
                if (Os.isFamily(Os.FAMILY_WINDOWS)) {
                    commandLine("cmd", "/c", "java", "-version")
                } else {
                    commandLine "bash", "-c", "java -version"
                }
                doLast {
                    exec {
                        if (Os.isFamily(Os.FAMILY_WINDOWS)) {
                            commandLine("cmd", "/c", "npm", "run", "build_android_common_bundle")
                        } else {
                            commandLine "bash", "-c", "npm run build_android_common_bundle"
                        }
                    }
                    exec {
                        if (Os.isFamily(Os.FAMILY_WINDOWS)) {
                            commandLine("cmd", "/c", "npm", "run", "build_android_diff_bundle")
                        } else {
                            commandLine "bash", "-c", "npm run build_android_diff_bundle"
                        }
                    }
                }
            } else {
                if (Os.isFamily(Os.FAMILY_WINDOWS)) {
                    commandLine("cmd", "/c", "java", "-version")
                } else {
                    commandLine "bash", "-c", "java -version"
                }
            }

            enabled config."bundleIn${targetName}" != null
                    ? config."bundleIn${targetName}"
                    : config."bundleIn${variant.buildType.name.capitalize()}" != null
                    ? config."bundleIn${variant.buildType.name.capitalize()}"
                    : targetName.toLowerCase().contains("release")
        }
        .....
        .....  
        .....
    }
}

```
但是如果你的工程集成Sentry之后，你会发现Sentry在打包时执行脚本会编译报错，下面我们来看下为什么会报错,`../../node_modules/@sentry/react-native/sentry.gradle`代码如下：
```
import org.apache.tools.ant.taskdefs.condition.Os

import java.util.regex.Matcher
import java.util.regex.Pattern

def config = project.hasProperty("sentryCli") ? project.sentryCli : [];

gradle.projectsEvaluated {
    def releases = extractReleasesInfo()

    if (config.flavorAware && config.sentryProperties) {
        throw new GradleException("Incompatible sentry configuration. " +
                "You cannot use both `flavorAware` and `sentryProperties`. " +
                "Please remove one of these from the project.ext.sentryCli configuration.")
    }

    if (config.sentryProperties instanceof String) {
        config.sentryProperties = file(config.sentryProperties)
    }

    if (config.sentryProperties) {
        if (!config.sentryProperties.exists()) {
            throw new GradleException("project.ext.sentryCli configuration defines a non-existant 'sentryProperties' file: " + config.sentryProperties.getAbsolutePath())
        }
        logger.info("Using 'sentry.properties' at: " + config.sentryProperties.getAbsolutePath())
    }

    if (config.flavorAware) {
        println "**********************************"
        println "* Flavor aware sentry properties *"
        println "**********************************"
    }

    // separately we then hook into the bundle task of react native to inject
    // sourcemap generation parameters.  In case for whatever reason no release
    // was found for the asset folder we just bail.
    def bundleTasks = tasks.findAll { task -> task.name.startsWith("bundle") && task.name.endsWith("JsAndAssets") && !task.name.contains("Debug") }
    bundleTasks.each { bundleTask ->
        def shouldCleanUp
        def sourcemapOutput
        def bundleOutput
        def props = bundleTask.getProperties()
        def reactRoot = props.get("workingDir")

        (shouldCleanUp, bundleOutput, sourcemapOutput) = forceSourceMapOutputFromBundleTask(bundleTask)

        // Lets leave this here if we need to debug
        // println bundleTask.properties
        //     .sort{it.key}
        //     .collect{it}
        //     .findAll{!['class', 'active'].contains(it.key)}
        //     .join('\n')

        def currentVariants = extractCurrentVariants(bundleTask, releases)
        if (currentVariants == null) return

        def variant = null
        def releaseName = null
        def versionCodes = new ArrayList<Integer>(currentVariants.size())

        currentVariants.each { key, currentVariant ->
            variant = currentVariant[0]
            releaseName = currentVariant[1]
            versionCodes.push(currentVariant[2])
        }

        def nameCliTask = "${bundleTask.name}_SentryUpload"
        def nameCleanup = "${bundleTask.name}_SentryUploadCleanUp"

        /** Upload source map file to the sentry server via CLI call. */
        def cliTask = tasks.create(name: nameCliTask, type: Exec) {
            description = "upload debug symbols to sentry"
            group = 'sentry.io'

            workingDir reactRoot

            def propertiesFile = config.sentryProperties
                    ? config.sentryProperties
                    : "$reactRoot/android/sentry.properties"

            if (config.flavorAware) {
                propertiesFile = "$reactRoot/android/sentry-${variant}.properties"
                project.logger.info("For $variant using: $propertiesFile")
            } else {
                environment("SENTRY_PROPERTIES", propertiesFile)
            }

            Properties sentryProps = new Properties()
            try {
                sentryProps.load(new FileInputStream(propertiesFile))
            } catch (FileNotFoundException e) {
                project.logger.info("file not found '$propertiesFile' for '$variant'")
            }
            def cliExecutable = sentryProps.get("cli.executable", "$reactRoot/node_modules/@sentry/cli/bin/sentry-cli")

            // fix path separator for Windows
            if (Os.isFamily(Os.FAMILY_WINDOWS)) {
                cliExecutable = cliExecutable.replaceAll("/", "\\\\")
            }

            //
            // based on:
            //   https://github.com/getsentry/sentry-cli/blob/master/src/commands/react_native_gradle.rs
            //
            def args = [cliExecutable]

            args.addAll(!config.logLevel ? [] : [
                    "--log-level", config.logLevel      // control verbosity of the output
            ])
            args.addAll(!config.flavorAware ? [] : [
                    "--url", sentryProps.get("defaults.url"),
                    "--auth-token", sentryProps.get("auth.token")
            ])
            args.addAll(["react-native", "gradle",
                         "--bundle", bundleOutput,           // The path to a bundle that should be uploaded.
                         "--sourcemap", sourcemapOutput,     // The path to a sourcemap that should be uploaded.
                         "--release", releaseName            // The name of the release to publish.
            ])
            args.addAll(!config.flavorAware ? [] : [
                    "--org", sentryProps.get("defaults.org"),
                    "--project", sentryProps.get("defaults.project")
            ])

            // The names of the distributions to publish. Can be supplied multiple times.
            versionCodes.each { versionCode -> args.addAll(["--dist", versionCode]) }

            project.logger.info("Sentry-CLI arguments: ${args}")

            def osCompatibility = Os.isFamily(Os.FAMILY_WINDOWS) ? ['cmd', '/c', 'node'] : []
            commandLine(*osCompatibility, *args)

            enabled true
        }

        /** Delete sourcemap files */
        def cliCleanUpTask = tasks.create(name: nameCleanup, type: Delete) {
            description = "clean up extra sourcemap"
            group = 'sentry.io'

            delete sourcemapOutput
            delete "$buildDir/intermediates/assets/release/index.android.bundle.map" // react native default bundle dir
        }

        // dependsOn, mustRunAfter, shouldRunAfter, doFirst, doLast, finalizedBy
        // bundleTask --> cliTask
        bundleTask.finalizedBy cliTask

        // register clean task extension
        cliCleanUpTask.onlyIf { shouldCleanUp }
        cliTask.finalizedBy cliCleanUpTask
    }
}

/** Compose lookup map of build variants - to - outputs. */
def extractReleasesInfo() {
    def releases = [:]

    android.applicationVariants.each { variant ->

        variant.outputs.each { output ->
            def versionCode = output.getVersionCode()
            def releaseName = "${variant.getApplicationId()}@${variant.getVersionName()}+${versionCode}"
            def variantName = variant.getName()
            def outputName = output.getName()
            if (releases[variantName] == null) {
                releases[variantName] = [:]
            }
            releases[variantName][outputName] = [outputName, releaseName, versionCode]
        }
    }

    return releases
}

/** Extract from arguments collection bundle and sourcemap files output names. */
static extractBundleTaskArguments(cmdArgs, Project project) {
    def bundleOutput = null
    def sourcemapOutput = null

    cmdArgs.eachWithIndex { String arg, int i ->
        if (arg == "--bundle-output") {
            bundleOutput = cmdArgs[i + 1]
            project.logger.info("--bundle-output: `${bundleOutput}`")
        } else if (arg == "--sourcemap-output") {
            sourcemapOutput = cmdArgs[i + 1]
            project.logger.info("--sourcemap-output param: `${sourcemapOutput}`")
        }
    }

    // Best thing would be if we just had access to the local gradle variables here:
    // https://github.com/facebook/react-native/blob/ff3b839e9a5a6c9e398a1327cde6dd49a3593092/react.gradle#L89-L97
    // Now, the issue is that hermes builds have a different pipeline:
    // `metro -> hermes -> compose-source-maps`, which then combines both intermediate sourcemaps into the final one.
    // In this function here, we only grep through the first `metro` step, which only generates an intermediate sourcemap,
    // which is wrong. We need the final one. Luckily, we can just generate the path from the `bundleOutput`, since
    // the paths seem to be well defined.

    // if sourcemapOutput is null, it means there's no source maps at all
    // if hermes is enabled and has intermediates folder, we need to fix paths
    // if hermes is disabled, sourcemapOutput is already ok
    def enableHermes = project.ext.react.get("enableHermes", false);
    project.logger.info("enableHermes: `${enableHermes}`")

    if (bundleOutput != null && sourcemapOutput != null && enableHermes) {
        // react-native < 0.60.1
        def pattern = Pattern.compile("(/|\\\\)intermediates\\1sourcemaps\\1react\\1")
        Matcher matcher = pattern.matcher(sourcemapOutput)
        // if its intermediates/sourcemaps/react then it should be generated/sourcemaps/react
        if (matcher.find()) {
            project.logger.info("sourcemapOutput has the wrong path, let's fix it.")
            // replacing from bundleOutput which is more reliable
            sourcemapOutput = bundleOutput.replaceAll("(/|\\\\)generated\\1assets\\1react\\1", "\$1generated\$1sourcemaps\$1react\$1") + ".map"
            project.logger.info("sourcemapOutput new path: `${sourcemapOutput}`")
        }
    }

    return [bundleOutput, sourcemapOutput]
}

/** Force Bundle task to produce sourcemap files if they are not pre-configured by user yet. */
def forceSourceMapOutputFromBundleTask(bundleTask) {
    def props = bundleTask.getProperties()
    def cmd = props.get("commandLine") as List<String>
    def cmdArgs = props.get("args") as List<String>
    def shouldCleanUp = false
    def bundleOutput = null
    def sourcemapOutput = null

    (bundleOutput, sourcemapOutput) = extractBundleTaskArguments(cmdArgs, project)

    if (sourcemapOutput == null) {
        sourcemapOutput = bundleOutput + ".map"

        cmd.addAll(["--sourcemap-output", sourcemapOutput])
        cmdArgs.addAll(["--sourcemap-output", sourcemapOutput])

        shouldCleanUp = true

        bundleTask.setProperty("commandLine", cmd)
        bundleTask.setProperty("args", cmdArgs)

        project.logger.info("forced sourcemap file output for `${bundleTask.name}` task")
    } else {
        project.logger.info("Info: used pre-configured source map files: ${sourcemapOutput}")
    }

    return [shouldCleanUp, bundleOutput, sourcemapOutput]
}

/** compose array with one item - current build flavor name */
static extractCurrentVariants(bundleTask, releases) {
    // examples: bundleLocalReleaseJsAndAssets, bundleYellowDebugJsAndAssets
    def pattern = Pattern.compile("bundle([A-Z][A-Za-z0-9_]+)JsAndAssets")

    def currentRelease = ""

    Matcher matcher = pattern.matcher(bundleTask.name)
    if (matcher.find()) {
        def match = matcher.group(1)
        currentRelease = match.substring(0, 1).toLowerCase() + match.substring(1)
    }

    def currentVariants = null
    releases.each { key, release ->
        if (key.equalsIgnoreCase(currentRelease)) {
            currentVariants = release
        }
    }

    return currentVariants
}

```
我们可以看到在执行上传Task之前，执行了`forceSourceMapOutputFromBundleTask(bundleTask)`方法，继续追踪这个方法，可以看到以下关键代码：
```
/** Extract from arguments collection bundle and sourcemap files output names. */
static extractBundleTaskArguments(cmdArgs, Project project) {
    def bundleOutput = null
    def sourcemapOutput = null

    cmdArgs.eachWithIndex { String arg, int i ->
        if (arg == "--bundle-output") {
            bundleOutput = cmdArgs[i + 1]
            project.logger.info("--bundle-output: `${bundleOutput}`")
        } else if (arg == "--sourcemap-output") {
            sourcemapOutput = cmdArgs[i + 1]
            project.logger.info("--sourcemap-output param: `${sourcemapOutput}`")
        }
    }

    if (bundleOutput != null && sourcemapOutput != null && enableHermes) {
        // react-native < 0.60.1
        def pattern = Pattern.compile("(/|\\\\)intermediates\\1sourcemaps\\1react\\1")
        Matcher matcher = pattern.matcher(sourcemapOutput)
        // if its intermediates/sourcemaps/react then it should be generated/sourcemaps/react
        if (matcher.find()) {
            project.logger.info("sourcemapOutput has the wrong path, let's fix it.")
            // replacing from bundleOutput which is more reliable
            sourcemapOutput = bundleOutput.replaceAll("(/|\\\\)generated\\1assets\\1react\\1", "\$1generated\$1sourcemaps\$1react\$1") + ".map"
            project.logger.info("sourcemapOutput new path: `${sourcemapOutput}`")
        }
    }

    return [bundleOutput, sourcemapOutput]
}
```
这段代码的意思是从上个依赖Task中取--bundle-output和--sourcemap-output的入参作为之前上传Task中使用到的bundleOutput和sourcemapOutput命令入参
```
args.addAll(["react-native", "gradle",
             "--bundle", bundleOutput,           // The path to a bundle that should be uploaded.
             "--sourcemap", sourcemapOutput,     // The path to a sourcemap that should be uploaded.
             "--release", releaseName            // The name of the release to publish.
])
```
由于我们之前改写了react.gralde中的打包脚本，所以这段逻辑就无法取到--bundle-output和--sourcemap-output的入参，致使打包编译失败，那么我们就去改写这段脚本来去适配Sentry上传脚本：
```
/** Extract from arguments collection bundle and sourcemap files output names. */
static extractBundleTaskArguments(Project project) {
    def bundleOutput = "${project.buildDir}/generated/assets/react/release/index.android.bundle"
    def sourcemapOutput = "${project.buildDir}/generated/sourcemaps/react/release/index.android.bundle.map"
    return [bundleOutput, sourcemapOutput]
}
```

