'use strict'
// var DEBUG = true;
var DEBUG = false

/**
 * simple JSONP support
 *
 *     JSONP.get('https://api.github.com/gists/1431613', function (data) { console.log(data); });
 *     JSONP.get('https://api.github.com/gists/1431613', {}, function (data) { console.log(data); });
 *
 * gist: https://gist.github.com/gists/1431613
 */
var JSONP = (function (document) {
  var requests = 0,
    callbacks = {}

  return {
    /**
     * makes a JSONP request
     *
     * @param {String} src
     * @param {Object} data
     * @param {Function} callback
     */
    get: function (src, data, callback) {
      // check if data was passed
      if (!arguments[2]) {
        callback = arguments[1]
        data = {}
      }

      // determine if there already are params
      src += src.indexOf('?') + 1 ? '&' : '?'

      var head = document.getElementsByTagName('head')[0],
        script = document.createElement('script'),
        params = [],
        requestId = requests,
        param

      // increment the requests
      requests++

      // create external callback name
      data.callback = 'JSONP.callbacks.request_' + requestId

      // set callback function
      callbacks['request_' + requestId] = function (data) {
        // clean up
        head.removeChild(script)
        delete callbacks['request_' + requestId]

        // fire callback
        callback(data)
      }

      // traverse data
      for (param in data) {
        params.push(param + '=' + encodeURIComponent(data[param]))
      }

      // generate params
      src += params.join('&')

      // set script attributes
      script.type = 'text/javascript'
      script.src = src

      // add to the DOM
      head.appendChild(script)
    },

    /**
     * keeps a public reference of the callbacks object
     */
    callbacks: callbacks
  }
})(document)

var queue = window.ana.q || []
// var trackerUrl = '//ana.dev/collect';
var trackerUrl = '//ana.dev'
var trackerCode = ''
var secondsInaYear = 31557600
var idleTimeoutInSeconds = 5
var sessionTimeoutInSeconds = 30 * 60
var defaultTimeoutInSeconds = secondsInaYear * 2
var user = {
  id: null
}
var commands = {
  trackPageview: trackPageview,
  setTrackerUrl: setTrackerUrl,
  setUserID: setUserID,
  setTrackingCode: setTrackingCode
}
var pixel
var pixelLoadedTimeOut
var vk = false //visitor key
var pk = uuid() //page view key
var pi //page id
var sk
var st
var ru
var stReported = 0
var addToSession = 0
var spvc = 0
var lastTracked = {}
var trackingLocation = document.location.toString()
var waitForVkTimeout = null

var REFERRAL_EXCLUSION_LIST = [
  'localhost',
  //  'staging.clientify.net',
  //  'staging2.clientify.net',
  'www.clientify.net',
  //  'clientify.net',
  'clientify.local'
]

// e=event
// vi=value int
// vf=value float
// vs=value string
// vm=value meta
// top=value time on page

// convert object to query string
function stringifyObject(json) {
  var keys = Object.keys(json)

  // console.log('stringifyObject')
  // console.log(json)

  // omit empty
  keys = keys.filter(function (k) {
    return json[k].length > 0
  })

  return (
    '?' +
    keys
      .map(function (k) {
        return encodeURIComponent(k) + '=' + encodeURIComponent(json[k])
      })
      .join('&')
  )
}

function setTrackerUrl(v) {
  trackerUrl = v
}

function setUserID(uid) {
  user.id = uid
  setCDays('uid', user.id, 2 * 365)
}

function setTrackingCode(tc) {
  trackerCode = tc
}

function onPixelLoaded(i, try_count) {
  var duration = 100
  var timeElapsed = duration * try_count

  clearTimeout(pixelLoadedTimeOut)

  if (timeElapsed > 30000) {
    return
  }

  if (!i.complete) {
    // console.log('pixel NOT completed:' + try_count);
    pixelLoadedTimeOut = setTimeout(function () {
      onPixelLoaded(i, try_count + 1)
    }, duration)
  } else {
    onPixelLoadedCallback()
  }
}
function onPixelLoadedCallback(i, try_count) {
  sk = getSk()
  trackPageviewTime()

  if (document.location.toString().indexOf('debug=on') > 0) {
    setC('debug', 'on', 60 * 60 * 24)
  } else if (document.location.toString().indexOf('debug=off') > 0) {
    setC('debug', 'off', 60 * 60 * 24)
  }

  var _debug = C('debug')

  if (typeof _debug !== 'undefined' && _debug === 'on') {
    DEBUG = true
  }

  if (DEBUG) {
    launchDebugPanel()
  }
}

function trackPageviewFinish() {
  if (vk === false || __empty(vk)) {
    // console.log('waiting for vk')
    waitForVkTimeout = setTimeout(trackPageviewFinish, 200)

    return
  } else {
    // console.log('vk READY')
    // console.log(vk)
    // alert("vk READY");
    // alert(vk);
    clearTimeout(waitForVkTimeout)

    var referrer = document.referrer

    // TODO: Implement this also on backend
    if (__notEmpty(referrer)) {
      var referrerLocation = parseURL(referrer)
      if (isInArray(REFERRAL_EXCLUSION_LIST, referrerLocation.hostname)) {
        return
      }
    }

    // get the path or canonical
    var path = location.pathname + location.search

    var canonical = document.querySelector('link[rel="canonical"]')
    if (canonical && canonical.href) {
      if (canonical.href.indexOf('https://') === 0) {
        path = canonical.href.substring(canonical.href.indexOf('/', 8)) || '/'
      } else {
        path = canonical.href.substring(canonical.href.indexOf('/', 7)) || '/'
      }
    }

    if (
      document.referrer.indexOf(location.protocol + '//' + location.host) === 0
    ) {
      referrer = ''
    } else {
      referrer = document.referrer
    }

    ru = referrer

    var d = {
      vk: vk,
      pk: pk,
      t: document.title,
      l: navigator.language,
      tc: trackerCode,
      h: location.host,
      p: path,
      tl: trackingLocation,
      sr: screen.width + 'x' + screen.height,
      ru: ru,
      rk: '',
      sk: getSk(),
      ats: addToSession.toFixed(2) + '',
      spvc: getSpvc(true) + ''
      // ua: ((window && window.navigator && window.navigator.userAgent) ? window.navigator.userAgent : EMPTY)
    }

    lastTracked = d
    // var vk; //visitor key
    // var pk; //page view key

    var i = new Image()
    i.src = trackerUrl + '/collect' + stringifyObject(d)
    i.alt = 'image'

    pixelLoadedTimeOut = setTimeout(function () {
      onPixelLoaded(i, 1)
    }, 100)

    //document.body.appendChild(i)
  }
}

function trackPageview() {
  // Respect "Do Not Track" requests
  if (navigator.DonotTrack == 1) {
    return
  }

  vk = getAnyKey('vk')
  waitForVkTimeout = setTimeout(trackPageviewFinish, 100)
}

// override global ana object
window.ana = function () {
  var args = [].slice.call(arguments)
  var c = args.shift()
  commands[c].apply(this, args)
}

// process existing queue
queue.forEach(function (i) {
  ana.apply(this, i)
})

// getSk returns the session key
function getSk() {
  var _sk = C('sk')

  if (__notEmpty(_sk)) {
    // console.log('Existing Session Key, renewing: ' + _sk);
    setC('sk', _sk, sessionTimeoutInSeconds)
  } else {
    _sk = uuid()
    setC('sk', _sk, sessionTimeoutInSeconds)
    sk = _sk
    // console.log('New Session Key: ' + sk);
  }

  return _sk
}

// getAnyKey returns a key from a cookie and renews expiration, if not found creates a uuid and saves it to a cookie
function getAnyKey(key, timeout) {
  if (__empty(timeout)) {
    timeout = defaultTimeoutInSeconds
  }

  var _k = C(key)

  if (__notEmpty(_k)) {
    // console.log('Existing Key, renewing: ' + _k)
    setC(key, _k, timeout)
  } else {
    // alert('creating new key');

    // example request
    // getCORS('http://tracking.local:2030/c', function (request) {
    //   var response = request.currentTarget.response || request.target.responseText;
    //   console.log(response);
    // });

    var url = trackerUrl + '/c'

    JSONP.get(url, function (data) {
      // alert(data.vk)
      if (__notEmpty(data.vk)) {
        _k = data.vk
      } else {
        _k = uuid()
      }

      vk = _k
      // alert(vk)
      setC(key, _k, timeout)
    })
  }

  return _k
}

//getSpvc returns session pageview count
function getSpvc(increase) {
  var _spvc = C('spvc')

  if (__notEmpty(_spvc)) {
    if (increase === true) {
      _spvc = parseInt(_spvc) + 1
    }

    setC('spvc', _spvc, sessionTimeoutInSeconds)
  } else {
    if (increase === true) {
      _spvc = 1
    } else {
      _spvc = 0
    }
    setC('spvc', _spvc, sessionTimeoutInSeconds)
  }

  spvc = _spvc

  return _spvc
}

function trackPageviewTime() {
  TimeMe.initialize({
    currentPageName: document.title,
    idleTimeoutInSeconds: idleTimeoutInSeconds
  })
  // TimeMe.startTimer("session-time");

  // TimeMe.startTimer("session-time");

  // TimeMe.startTimer("pageview");

  var callCount = 0
  TimeMe.callWhenUserLeaves(function () {
   /*  console.log(
      'The user is not currently viewing the page! callCount: ' +
      callCount +
      '\nsk:' +
      getSk()
    ) */
    getSk();
    callCount = callCount + 1
    // e=event
    // vk=visitor key
    // pk=page view key
    // pi=page id
    // vi=value int
    // vf=value float
    // vs=value string
    // vm=value meta
    // top=value time on page

    var top = TimeMe.getTimeOnCurrentPageInSeconds()
    var d = {
      vf: top.toFixed(2)
    }

    addToSession = top - stReported

    trackEvent('___time-on-page', d)

    stReported = parseFloat(top)
  }, 10000)
}

function trackEvent(event, values) {
  var d = {
    e: event,
    tc: trackerCode,
    top: TimeMe.getTimeOnCurrentPageInSeconds().toFixed(2)
  }

  if (__notEmpty(vk)) {
    d.vk = vk
  }

  if (__notEmpty(pk)) {
    d.pk = pk
  }

  if (__notEmpty(user.id)) {
    d.uid = user.id + ''

    if (!checkC('uids')) {
      console.log('uids not set yet')
    } else {
      console.log('uids already set')
    }
  }

  if (__notEmpty(trackerCode)) {
    d.tc = trackerCode + ''
  }

  d.ats = addToSession.toFixed(2) + ''
  d.sk = getSk()
  d.spvc = getSpvc(false)

  d = __mA(d, values)

  var i = new Image()
  i.src = trackerUrl + '/track' + stringifyObject(d)
  i.alt = 'image'

  //console.log(i.src)
  //document.body.appendChild(i)
}

// https://plainjs.com/javascript/ajax/making-cors-ajax-get-requests-54/
function postCORS(url, success) {
  var xhr = new XMLHttpRequest()
  if (!('withCredentials' in xhr)) xhr = new XDomainRequest() // fix IE8/9
  xhr.withCredentials = false
  xhr.open('GET', url)
  xhr.onload = success
  xhr.send()
  return xhr

  // // For cross-origin requests, some simple logic
  // // to determine if XDomainReqeust is needed.
  // if (new XMLHttpRequest().withCredentials === undefined) {
  //   var xdr = new XDomainRequest();
  //   xdr.open('GET', url);
  //   xdr.send('sometext');
  //   return xhr;
  // }
}

// function getCORS (url, success) {
//   var xhr = new XMLHttpRequest();
//   if (!('withCredentials' in xhr)) xhr = new XDomainRequest(); // fix IE8/9
//   xhr.open('GET', url);
//   xhr.onload = success;
//   xhr.send();
//   return xhr;
// }

// http://stackoverflow.com/questions/43449788/how-do-i-merge-two-dictionaries-in-javascript
function __mA(a, b) {
  var merged = [a, b].reduce(function (r, o) {
    Object.keys(o).forEach(function (k) {
      r[k] = o[k]
    })
    return r
  }, {})

  return merged
}

function __empty(s) {
  if (__notEmpty(s)) {
    return false
  }

  return true
}

function __notEmpty(s) {
  if (s === undefined) {
    s = ''
  } else {
    s += ''
  }

  if (s.length > 0) {
    return true
  }

  return false
}
// https://stackoverflow.com/questions/5639346/what-is-the-shortest-function-for-reading-a-cookie-by-name-in-javascript
function C(k) {
  return (document.cookie.match('(^|; )' + k + '=([^;]*)') || 0)[2]
}

// https://www.w3schools.com/js/js_cookies.asp
function setC(cname, cvalue, exsec) {
  var d = new Date()
  d.setTime(d.getTime() + exsec * 1000)
  // d.setTime(d.getTime() + (exdays * 24 * 60 * 60 * 1000));
  var expires = 'expires=' + d.toUTCString()
  document.cookie =
    cname + '=' + cvalue + ';' + expires + ';SameSite=None;Secure;path=/'
}
// TODO: Cleanup these cookie stuff

function setCDays(cname, cvalue, exdays) {
  // d.setTime(d.getTime() + (exdays * 24 * 60 * 60 * 1000));
  setC(cname, cvalue, exdays * 60 * 60 * 24)
}

// https://www.w3schools.com/js/js_cookies.asp
function checkC(k) {
  var v = C(k)
  if (__notEmpty(v)) {
    return true
  }

  return false
}

// https://gist.github.com/jcxplorer/823878
function uuid() {
  var uuid = '',
    i,
    random
  for (i = 0; i < 32; i++) {
    random = (Math.random() * 16) | 0

    if (i == 8 || i == 12 || i == 16 || i == 20) {
      uuid += '-'
    }
    uuid += (i == 12 ? 4 : i == 16 ? (random & 3) | 8 : random).toString(16)
  }
  return uuid
}

// https://stackoverflow.com/questions/14461450/check-if-string-inside-an-array-javascript
function isInArray(values, value) {
  return values.indexOf(value.toLowerCase()) > -1
}

function parseURL(url) {
  var parser = document.createElement('a')
  parser.href = url
  return parser

  // parser.protocol; // => "http:"
  // parser.hostname; // => "example.com"
  // parser.port;     // => "3000"
  // parser.pathname; // => "/pathname/"
  // parser.search;   // => "?search=test"
  // parser.hash;     // => "#hash"
  // parser.host;     // => "example.com:3000"
}

// timeme.min.js
// http://timemejs.com/
; (function () {
  !(function (e, t) {
    return 'undefined' != typeof module && module.exports
      ? (module.exports = t())
      : 'function' == typeof define && define.amd
        ? void define([], function () {
          return (e.TimeMe = t())
        })
        : (e.TimeMe = t())
  })(this, function () {
    var e = {
      startStopTimes: {},
      idleTimeoutMs: 3e4,
      currentIdleTimeMs: 0,
      checkStateRateMs: 250,
      active: !1,
      idle: !1,
      currentPageName: 'default-page-name',
      timeElapsedCallbacks: [],
      userLeftCallbacks: [],
      userReturnCallbacks: [],
      trackTimeOnElement: function (t) {
        var n = document.getElementById(t)
        n &&
          (n.addEventListener('mouseover', function () {
            e.startTimer(t)
          }),
            n.addEventListener('mousemove', function () {
              e.startTimer(t)
            }),
            n.addEventListener('mouseleave', function () {
              e.stopTimer(t)
            }),
            n.addEventListener('keypress', function () {
              e.startTimer(t)
            }),
            n.addEventListener('focus', function () {
              e.startTimer(t)
            }))
      },
      getTimeOnElementInSeconds: function (t) {
        var n = e.getTimeOnPageInSeconds(t)
        return n ? n : 0
      },
      startTimer: function (t) {
        if ((t || (t = e.currentPageName), void 0 === e.startStopTimes[t]))
          e.startStopTimes[t] = []
        else {
          var n = e.startStopTimes[t],
            i = n[n.length - 1]
          if (void 0 !== i && void 0 === i.stopTime) return
        }
        e.startStopTimes[t].push({ startTime: new Date(), stopTime: void 0 }),
          (e.active = !0)
      },
      stopAllTimers: function () {
        for (var t = Object.keys(e.startStopTimes), n = 0; n < t.length; n++)
          e.stopTimer(t[n])
      },
      stopTimer: function (t) {
        t || (t = e.currentPageName)
        var n = e.startStopTimes[t]
        void 0 !== n &&
          0 !== n.length &&
          (void 0 === n[n.length - 1].stopTime &&
            (n[n.length - 1].stopTime = new Date()),
            (e.active = !1))
      },
      getTimeOnCurrentPageInSeconds: function () {
        return e.getTimeOnPageInSeconds(e.currentPageName)
      },
      getTimeOnPageInSeconds: function (t) {
        var n = e.getTimeOnPageInMilliseconds(t)
        return void 0 === n ? void 0 : e.getTimeOnPageInMilliseconds(t) / 1e3
      },
      getTimeOnCurrentPageInMilliseconds: function () {
        return e.getTimeOnPageInMilliseconds(e.currentPageName)
      },
      getTimeOnPageInMilliseconds: function (t) {
        var n = 0,
          i = e.startStopTimes[t]
        if (void 0 !== i) {
          for (var s = 0, o = 0; o < i.length; o++) {
            var r = i[o].startTime,
              a = i[o].stopTime
            void 0 === a && (a = new Date())
            var d = a - r
            s += d
          }
          return (n = Number(s))
        }
      },
      getTimeOnAllPagesInSeconds: function () {
        for (
          var t = [], n = Object.keys(e.startStopTimes), i = 0;
          i < n.length;
          i++
        ) {
          var s = n[i],
            o = e.getTimeOnPageInSeconds(s)
          t.push({ pageName: s, timeOnPage: o })
        }
        return t
      },
      setIdleDurationInSeconds: function (t) {
        var n = parseFloat(t)
        if (isNaN(n) !== !1)
          throw {
            name: 'InvalidDurationException',
            message: 'An invalid duration time (' + t + ') was provided.'
          }
        return (e.idleTimeoutMs = 1e3 * t), this
      },
      setCurrentPageName: function (t) {
        return (e.currentPageName = t), this
      },
      resetRecordedPageTime: function (t) {
        delete e.startStopTimes[t]
      },
      resetAllRecordedPageTimes: function () {
        for (var t = Object.keys(e.startStopTimes), n = 0; n < t.length; n++)
          e.resetRecordedPageTime(t[n])
      },
      resetIdleCountdown: function () {
        e.idle && e.triggerUserHasReturned(),
          (e.idle = !1),
          (e.currentIdleTimeMs = 0)
      },
      callWhenUserLeaves: function (e, t) {
        this.userLeftCallbacks.push({ callback: e, numberOfTimesToInvoke: t })
      },
      callWhenUserReturns: function (e, t) {
        this.userReturnCallbacks.push({ callback: e, numberOfTimesToInvoke: t })
      },
      triggerUserHasReturned: function () {
        if (!e.active)
          for (var t = 0; t < this.userReturnCallbacks.length; t++) {
            var n = this.userReturnCallbacks[t],
              i = n.numberOfTimesToInvoke
              ; (isNaN(i) || void 0 === i || i > 0) &&
                ((n.numberOfTimesToInvoke -= 1), n.callback())
          }
        e.startTimer()
      },
      triggerUserHasLeftPage: function () {
        if (e.active)
          for (var t = 0; t < this.userLeftCallbacks.length; t++) {
            var n = this.userLeftCallbacks[t],
              i = n.numberOfTimesToInvoke
              ; (isNaN(i) || void 0 === i || i > 0) &&
                ((n.numberOfTimesToInvoke -= 1), n.callback())
          }
        e.stopAllTimers()
      },
      callAfterTimeElapsedInSeconds: function (t, n) {
        e.timeElapsedCallbacks.push({
          timeInSeconds: t,
          callback: n,
          pending: !0
        })
      },
      checkState: function () {
        for (var t = 0; t < e.timeElapsedCallbacks.length; t++)
          e.timeElapsedCallbacks[t].pending &&
            e.getTimeOnCurrentPageInSeconds() >
            e.timeElapsedCallbacks[t].timeInSeconds &&
            (e.timeElapsedCallbacks[t].callback(),
              (e.timeElapsedCallbacks[t].pending = !1))
        e.idle === !1 && e.currentIdleTimeMs > e.idleTimeoutMs
          ? ((e.idle = !0), e.triggerUserHasLeftPage())
          : (e.currentIdleTimeMs += e.checkStateRateMs)
      },
      visibilityChangeEventName: void 0,
      hiddenPropName: void 0,
      listenForVisibilityEvents: function () {
        'undefined' != typeof document.hidden
          ? ((e.hiddenPropName = 'hidden'),
            (e.visibilityChangeEventName = 'visibilitychange'))
          : 'undefined' != typeof doc.mozHidden
            ? ((e.hiddenPropName = 'mozHidden'),
              (e.visibilityChangeEventName = 'mozvisibilitychange'))
            : 'undefined' != typeof document.msHidden
              ? ((e.hiddenPropName = 'msHidden'),
                (e.visibilityChangeEventName = 'msvisibilitychange'))
              : 'undefined' != typeof document.webkitHidden &&
              ((e.hiddenPropName = 'webkitHidden'),
                (e.visibilityChangeEventName = 'webkitvisibilitychange')),
          document.addEventListener(
            e.visibilityChangeEventName,
            function () {
              document[e.hiddenPropName]
                ? e.triggerUserHasLeftPage()
                : e.triggerUserHasReturned()
            },
            !1
          ),
          window.addEventListener('blur', function () {
            e.triggerUserHasLeftPage()
          }),
          window.addEventListener('focus', function () {
            e.triggerUserHasReturned()
          }),
          document.addEventListener('mousemove', function () {
            e.resetIdleCountdown()
          }),
          document.addEventListener('keyup', function () {
            e.resetIdleCountdown()
          }),
          document.addEventListener('touchstart', function () {
            e.resetIdleCountdown()
          }),
          window.addEventListener('scroll', function () {
            e.resetIdleCountdown()
          }),
          setInterval(function () {
            e.checkState()
          }, e.checkStateRateMs)
      },
      websocket: void 0,
      websocketHost: void 0,
      setUpWebsocket: function (t) {
        if (window.WebSocket && t) {
          var n = t.websocketHost
          try {
            ; (e.websocket = new WebSocket(n)),
              (window.onbeforeunload = function (n) {
                e.sendCurrentTime(t.appId)
              }),
              (e.websocket.onopen = function () {
                e.sendInitWsRequest(t.appId)
              }),
              (e.websocket.onerror = function (e) {
                console &&
                  console.log('Error occurred in websocket connection: ' + e)
              }),
              (e.websocket.onmessage = function (e) {
                console && console.log(e.data)
              })
          } catch (i) {
            console &&
              console.error('Failed to connect to websocket host.  Error:' + i)
          }
        }
        return this
      },
      websocketSend: function (t) {
        e.websocket.send(JSON.stringify(t))
      },
      sendCurrentTime: function (t) {
        var n = e.getTimeOnCurrentPageInMilliseconds(),
          i = {
            type: 'INSERT_TIME',
            appId: t,
            timeOnPageMs: n,
            pageName: e.currentPageName
          }
        e.websocketSend(i)
      },
      sendInitWsRequest: function (t) {
        var n = { type: 'INIT', appId: t }
        e.websocketSend(n)
      },
      initialize: function (t) {
        var n = e.idleTimeoutMs || 30,
          i = e.currentPageName || 'default-page-name',
          s = void 0
        t &&
          ((n = t.idleTimeoutInSeconds || n),
            (i = t.currentPageName || i),
            (s = t.websocketOptions)),
          e
            .setIdleDurationInSeconds(n)
            .setCurrentPageName(i)
            .setUpWebsocket(s)
            .listenForVisibilityEvents(),
          e.startTimer()
      }
    }
    return e
  })
}.call(this))

//  ******************************************************* //
//  DEBUG FUNCTIONS, MUST BE REMOVED WHEN DEPLOYING TO PROD //
//  ******************************************************* //

function fixFormURLS() {
  var formElement = document.getElementById('f25a5fab_id')
  var currentAction = formElement.getAttribute('action')
  var newAction
  if (
    trackerUrl === 'https://analytics.clientify.net' &&
    currentAction.indexOf('https://staging.clientify.net') == 0
  ) {
    // alert('neet to fix')
    newAction = currentAction.replace(
      'https://staging.clientify.net',
      'https://clientify.net'
    )
    // alert(newAction)
    formElement.setAttribute('action', newAction)
    document
      .querySelector('[name="form_action"]')
      .setAttribute('value', newAction)
    fixFormSubmit()
  }
}

function fixFormSubmit() {
  form = document.getElementById('f25a5fab_id')

  if (!form) {
    form = document.querySelector('form[role="form"]')
  }
  var formAction = document.querySelector('input[name="form_action"]').value

  var errorMessage = 'Error submiting the form'

  form.setAttribute('action', formAction)

  form.onsubmit = function (e) {
    // Stop the regular form submission
    e.preventDefault()

    // Collect the form data while iterating over the inputs
    var data = {}
    for (var i = 0, ii = form.length; i < ii; ++i) {
      var input = form[i]
      if (input.name) {
        // data.append(input.id, input.value);
        data[input.name] = input.value
      }
    }

    data.visitor_key = vk

    // Construct an HTTP request
    var xhr = new XMLHttpRequest()
    // xhr.open(form.method, form.action, true);
    xhr.open('POST', formAction, true)

    xhr.setRequestHeader(
      'Accept',
      'application/x-www-form-urlencoded; charset=utf-8'
    )
    xhr.setRequestHeader(
      'Content-Type',
      'application/x-www-form-urlencoded; charset=UTF-8'
    )

    // Send the collected data as JSON
    xhr.send(param(data))

    document.querySelector('#f25a5fab_id').style.opacity = 0.5

    // Callback function
    xhr.onloadend = function (response) {
      document.querySelector('#f25a5fab_id').style.opacity = 1
      if (response.target.status === 0 || response.target.status === 400) {
        errorMsg.innerText = errorMessage
        errorMsg.style.display = 'block'
        successMsg.style.display = 'none'
      } else if (response.target.status === 200) {
        // Success
        var responseObj = JSON.parse(xhr.responseText)

        var errorMsg = document.getElementById('form_error_id')
        var successMsg = document.getElementById('form_success_id')

        if (responseObj.success) {
          // console.log(responseObj.tracking_pixel)
          var i = new Image()
          i.src = responseObj.tracking_pixel
          i.alt = 'image'

          // pixelLoadedTimeOut = setTimeout(function () {
          //   onPixelLoaded(i, 1);
          // }, 100);

          //document.body.appendChild(i)

          form.reset()
          errorMsg.style.display = 'none'
          if (responseObj.success_option === 'message') {
            successMsg.innerText = responseObj.success_text
            successMsg.style.display = 'block'
          } else if (
            responseObj.success_option === 'redirection' ||
            responseObj.success_option === 'landingpage'
          ) {
            document.location = responseObj.success_text
          }
        } else {
          errorMsg.innerText = errorMessage
          errorMsg.style.display = 'block'
          successMsg.style.display = 'none'
        }
      }
    }
  }
}

if (document.getElementById('f25a5fab_id')) {
  fixFormURLS()
}

window.TRACKER_LIB_LOADED = true

function _debug__TrackEvent() {
  var e = document.getElementById('_debug__select_event_id')
  var value = e.options[e.selectedIndex].value
  var text = e.options[e.selectedIndex].text

  if (value === '') {
    alert('Debes selecionar un evento')
    return
  }

  trackEvent(value, {})
}

function launchDebugPanel() {
  var p = document.createElement('div')
  p.style = `
  background: rgba(51, 122, 183, 0.9); 
  position: fixed;  
  width: 500px;  
  right: 10px;  
  bottom: 10px; 
  z-index: 99999;   
  padding: 10px; 
  color: #fff; 
  font-family: **sans-serif;`

  p.innerHTML += `
  <div style="font-weight: bolder; font-size:16px; color: #f9eb1d">Clientify Tracking Debug Panel</div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Tracking Code: <span id="_debug_panel_tc" ></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Unique Visitor Key: <span id="_debug_panel_vk" ></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Pageview key: <span id="_debug_panel_pk" ></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Pageview Duration: <span id="id_sec"></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Session Key: <span id="id_sk"></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Session Duration (added from this page): <span id="id_sk_time"></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Session Duration (new difference just added): <span id="id_sk_diff_time"></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Session Pageview Count: <span id="id_spvc"></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Host: <span id="id_host"></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Path: <span id="id_path"></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px">Refferal: <span id="id_referral"></span></div>
  <div style="font-weight: bolder; font-size:14px; margin-top: 10px; border:solid #fff 1px; padding:10px; color: f9eb1d">Test Custom Event Tracking
    <div style="margin-top:15px">
      <select id="_debug__select_event_id" style="background: black; padding: 3px 10px 3px 10px;">
        <option value="">Selecciona un evento</option>
        <option value="cold-lead"> - Cold lead</option>
        <option value="warm-lead"> - Warm Lead</option>
        <option value="hot-lead"> - Hot lead</option>
        <option value="client"> - Cliente</option>
        <option value="lost-lead"> - Lost lead</option>
        <option value="not-qualified-lead"> - Not qualified</option>
        <option value="other"> - Cliente</option>
      </select>
      <button onclick="_debug__TrackEvent()" type="button" style="border: 1px solid #fff; padding: 5px 10px 5px 10px; color: #fff; background: black;">Send Event</button>
    </div>  
  </div>

  
  `

  document.body.appendChild(p)

  setInterval(function () {
    var currentTime = TimeMe.getTimeOnCurrentPageInSeconds().toFixed(2)

    document.getElementById('_debug_panel_tc').innerHTML = trackerCode
    document.getElementById('_debug_panel_vk').innerHTML = vk
    document.getElementById('_debug_panel_pk').innerHTML = pk
    document.getElementById('id_sec').innerHTML = currentTime
    document.getElementById('id_sk').innerHTML = sk
    document.getElementById('id_sk_time').innerHTML = stReported.toFixed(2)
    document.getElementById('id_sk_diff_time').innerHTML = addToSession.toFixed(
      2
    )
    document.getElementById('id_spvc').innerHTML = getSpvc(false)
    document.getElementById('id_host').innerHTML = lastTracked.h
    document.getElementById('id_path').innerHTML = lastTracked.p
    document.getElementById('id_referral').innerHTML = ru
  }, 300)
}
