/* Copyright (c) 2008-2020, Quantcast Corp. https://www.quantcast.com/legal/license */
/* global window */
(function(window) {
    function RequireDependencyError(message) {
        Error.apply(this);
        this.name = "RequireDependencyError";
        this.message = (message || "");
    }

    // Required for compatiblity with Error types.
    RequireDependencyError.prototype = Error.prototype;

    var amd = {};

    // These guards against multiple execution are likely unwarranted.
    var definitions = {};

    /**
     * Require some things that have been defined.
     *
     * @param {Array} requires An array of strings defining the dependencies of
     * code being executed.
     * @param {Function} callback The code to be executed.
     */
    amd.require = function(dependencies, callback) {
        if (typeof dependencies === "function") {
            callback = dependencies;
            dependencies = [];
        }

        var args = [];

        for (var i = 0; i < dependencies.length; i++) {
            var module = dependencies[i];

            if (definitions.hasOwnProperty(module)) {
                args[i] = definitions[module];
            }
            else {
                throw new RequireDependencyError(
                    "No module named " + module + " has been defined"
                );
            }
        }

        return callback.apply({}, args);
    };

    // Array polyfills. These are not the official polyfills, they are much
    // smaller. They also pay for themselves. This funky polyfill trick is
    // intended to help minimization by hoisting the commonly used variables.
    //
    // Might be unnecessary, but whatever.
    var array = Array.prototype;

    var available = function(prototype, name) {
        return typeof prototype[name] === "function";
    };

    var map = "map", forEach = "forEach", reduce = "reduce", indexOf = "indexOf";

    if (!available(array, map)) {
        array[map] = function(f, o) {
            var result = [];

            if (!o) {
                o = this;
            }

            for (var i = 0; i < this.length; i++) {
                result[i] = f.call(o, this[i], i, this);
            }

            return result;
        };
    }

    if (!available(array, forEach)) {
        array[forEach] = array[map];
    }

    if (!available(array, reduce)) {
        array[reduce] = function(reducer, accumulator) {
            var i = 0;

            if (typeof accumulator === "undefined") {
                accumulator = this[i++];
            }

            for (; i < this.length; i++) {
                accumulator = reducer.call(this, accumulator, this[i], i, this);
            }

            return accumulator;
        };
    }

    if (!available(array, indexOf)) {
        array[indexOf] = function(element) {
            for (var i = 0; i < this.length; i++) {
                if (this[i] == element) {
                    return i;
                }
            }

            return -1;
        };
    }

    /**
     * Define some things.
     *
     * <p>Similar to {@link amd.require()}, this function will load
     * dependencies and execute the given callback.  The callback however is
     * stored as the definition under the name given, which can be used by
     * future calls to {@link amd.require()} or {@link amd.define()}.</p>
     *
     * <p>This function is <em>idempotent</em>, so multiple calls to not
     * execute the module factory callback.</p>
     *
     * @param {String} name The name of the module.
     * @param {Array} dependencies The dependencies of the code to be executed.
     * @param {Function} callback The constructor of this module.
     */
    amd.define = function(name, dependencies, callback) {
        if (!definitions.hasOwnProperty(name)) {
            definitions[name] = amd.require(dependencies, callback);
        }
    };

    with (amd) {
define('quant/now',[], function() {
    /**
     * Return the current time in epoch millis.
     */
    return function now() {
        return new Date().getTime();
    };
});

define('quant/origin',[ "quant/now" ], function(now) {
    var COOKIE = "_dlt=";

    /**
     * Determine the most permissive origin of a document.
     *
     * <p>In order to determine the correct domain to use as origin attempt to
     * set a cookie on the most permissive domain available. This defers
     * implementation of domain policy to the browser, allowing the browser to
     * tell us how permissive we can be by attempting to create a test cookie
     * on increasingly restrictive domains until one actually persists.
     * Delete's the test cookie when finished.</p>
     *
     * @param document The document where the cookies are to be set.
     * @returns The domain to be used for setting cookies.
     */
    return function origin(document) {
        var domain = document.domain || "";
        var expired = new Date(0).toUTCString();
        var tomorrow = new Date(now() + (24 * 60 * 60 * 1000) * 1).toUTCString();

        var levels = domain.split(".");
        var testDomain = "";

        // Only ever try 2nd level and higher domains. There is a subtle bug
        // where for some reason the `_dlt=1` cookie is present on the first
        // pass, likely due to execution terminating between setting and
        // deleting the cookie, or other browser misbehaviors.
        for (var i = 2; i <= levels.length; i++) {
            testDomain = levels.slice(-i).join(".");

            var testCookie = COOKIE + "1; path=/; domain=" + testDomain +
                "; expires=" + tomorrow;

            document.cookie = testCookie;

            if (/_dlt=1\b/.test(document.cookie)) {
                document.cookie = COOKIE + "; path=/; domain=" + testDomain + "; expires=" + expired;
                return testDomain;
            } 
        }

        // expires any remaining cookie in case all our tried domains have failed
        document.cookie = COOKIE + "; path=/; domain=" + testDomain + "; expires=" + expired;
        return domain;
    };
});

define('quant/windows',[], function() {
    

    /**
     * @constructor
     * Tests for windows.
     *
     * <p>This is a utility class which attempts to solve a handful of
     * use-cases, namely finding the top-most window that is *accessible*.</p>
     *
     * @param window The <code>window</code> global object for this script.
     * @param top The <code>window.top</code> object for this script.
     */
    return function Windows(window, top) {
        if (typeof window === "undefined") {
            throw new Error("window many not be undefined");
        }

        if (typeof top === "undefined") {
            throw new Error("top may not be undefined");
        }
        // Work around for an IE8 bug where window.top is often not the same
        // object every time.
        // https://stackoverflow.com/questions/4850978/ie-bug-window-top-false
        top = top.self;

        /**
         * The frame-depth of this window.
         *
         * <p>The number of frames this instance of the tag is located in.</p>
         *
         * @type Number
         */
        this.depth = 0;

        var ancestor = window.self;

        /**
         * The top most accessible window.
         *
         * @type Window
         */
        this.top = ancestor;

        while (ancestor !== top) {
            ancestor = ancestor.parent.self;

            try {
                // Just try to access the window.location, if we can get at it
                // and it is not undefined this is all the information we
                // actually want/need.
                if (ancestor.location.href) {
                    this.url = ancestor.location.href;
                    this.top = ancestor;

                }
            }
            catch (e) {
                /* Ignore the error as opposed to responding to it, because
                 * webkit might not throw one. */
            }

            this.depth++;
        }

        /**
         * Find parent of a locator frame.
         *
         * <p>It is a common feature to insert a frame called a locator in
         * order to advertise to other frames what the target of a post-message
         * API should be. It is generally the parent of the frame with that
         * name.</p>
         *
         * @param name The name of the locator frame to search for
         * @return The parent of the locator frame.
         */
        this.locate = function(name) {
            var frame = window;

            for (;;) {
                try {
                    if (name in frame.frames) {
                        return frame;
                    }
                }
                catch(error) {
                }

                if (frame === top) {
                    break;
                }

                frame = frame.parent.self;
            }
        };
    };
});

/* global window */
define(
'quant/log',[],function () {
    'use strict';

    function Logger (loader, hostname) {

        /**
         * Are we in debug mode?
         *
         * <p>This will be true if the string `qcdbgc=1` is present in the URL
         * that is loaded. You can turn it on by loading
         * `http://www.buzzfeed.com/#qcdbgc=1`.</p>
         */
        this.isDebug = /qcdbgc=1$/.test(window.location.toString());

        var timestamp = function () {
            return new Date().toString();
        };
        var write = function (level, args) {
            // Some browsers don't always have a console, namely IE.
            if(typeof console !== 'undefined') {
                console.log.apply(console, [level + ' ' + timestamp()].concat([].slice.call(args)));
            }
        };

        this.error = function () {
            write('ERROR', arguments);
        };

        this.debug = function () {
            if (this.isDebug) {
                write('DEBUG', arguments);
            }
        };
    }

    return Logger;
});

define('quant/ready',[],function() {
    'use strict';

    function ReadyHandler() {
        var ready = false;
        var callbacks = [];

        if (document.readyState in { 'complete': true, 'interactive': true }) {
            ready = true;
        }

        var handleReady = function () {
            ready = true;
            while (callbacks.length > 0) {
                callbacks.shift()();
            }
        };

        if (document.addEventListener) {
            document.addEventListener('DOMContentLoaded', handleReady, false);
            window.addEventListener('load', handleReady, false);
        }
        else if (document.attachEvent) {
            document.attachEvent('onreadystatechange', handleReady, false);
            window.attachEvent('onload', handleReady);
        }

        /**
         * Enqueue or execute this function.
         *
         * <p>Until the window is in an "ready" state (i.e. loading is complete
         * and so we are prepared to ammend additional resources) enqueue the
         * tasks which are passed to this function. Once this transition is
         * handled, execute this code immediately (this is effectively the same
         * as a promise, maybe it should be on a promise).
         */
        this.ready = function (callback) {
            if (ready) {
                callback();
            }
            else {
                callbacks.push(callback);
            }
        };
    }

    var readyHandler = new ReadyHandler();

    return readyHandler.ready;
});

define('quant/promise',[],function() {
    /**
     * Smaller implementation than ayepromise, though it is not fully A+
     * compatible. resolve, reject, `then()` and `catch()` work fine however,
     * as does binding and other basic properties. It also executes eagerly, as
     * opposed to guaranteeing asynchronous execution. Keep this in mind!
     *
     * There should probably not be very many polyfils in this codebase, but
     * promises are particularly useful for anything which may be asynchronous
     * and there's plenty of such behavior with the consent and lighttpd
     * integrations.
     *
     * This does depend on the forEach, map, and reduce polyfills for
     * Array.prototype.
     */
    var PENDING = 0, RESOLVED = 1, REJECTED = 2;

    // Let's just get the primary working without bothering about the bind
    // function. This is a composable-only promise for now.
    var thenable = function(value) {
        return typeof value === "object" &&
            "then" in value &&
            typeof value.then === "function";
    };

    var resolved = function(value) {
        if (thenable(value)) {
            return value;
        }

        return {
            then: function(callback) {
                return resolved(callback(value));
            },
            catch: function(callback) {
                return this;
            }
        };
    };

    var rejected = function(reason) {
        return {
            then: function(callback) {
                return this;
            },
            catch: function(callback) {
                return resolved(callback(reason));
            }
        };
    };

    function defer (callback) {
        var watchers = [];
        var result;
        var reason;
        var state = PENDING;

        // NOOP
        var passthrough = function(o) {
            return o;
        };

        var compose = function(apply, resolve, reject, value) {
            try {
                var composed = apply(value);

                if (thenable(composed)) {
                    composed.then(resolve);
                    composed.catch(reject);
                }
                else {
                    resolve(composed);
                }
            }
            catch (error) {
                reject(error);
            }
        };

        var resolve = function(value) {
            result = value;

            state = RESOLVED;

            watchers.forEach(function(watcher) {
                watcher.push(value);
                compose.apply(0, watcher);
            });
        };

        var reject = function(error) {
            reason = error;

            state = REJECTED;

            watchers.forEach(function(watcher) {
                watcher[REJECTED](error);
            });
        };

        var handle = function(apply, resolve, reject) {
            return function (error) {
                compose(apply, resolve, reject, error);
            };
        };

        try {
            callback(resolve, reject);
        }
        catch (exception) {
            reject(exception);
        }

        return {
            then: function(callback) {
                switch (state) {
                    case PENDING:
                        return new defer(function(resolve, reject) {
                            watchers.push([ callback, resolve, reject ]);
                        });
                    case RESOLVED:
                        return resolved(callback(result));
                    case REJECTED:
                        return rejected(reason);
                }
            },
            catch: function(callback) {
                switch (state) {
                    case PENDING:
                        return new defer(function(resolve, reject) {
                            // Catch is a little special, it attaches a promise
                            // which converts a failure to a resolution after
                            // applying a function on the error, so it binds to
                            // the compose operation. Object works as a cheap
                            // NOOP passthrough.
                            watchers.push([ passthrough, resolve,
                                handle(callback, resolve, reject) ]);
                        });
                    case RESOLVED:
                        return resolved(result);
                    case REJECTED:
                        return resolved(callback(reason));
                }
            }
        };
    }

    /**
     * Construct a resolved promise.
     *
     * <p>Given a value, produce a promise which is already resolved and
     * immediately available for composition. Ignores recovery attempts,
     * returning the same promise.</p>
     *
     * @param value The resolved value of this promise.
     */
    defer.resolve = resolved;

    /**
     * Construct a rejected promise.
     *
     * <p>Given an error (or error-like value), produce a promise which is
     * already rejected and immediately available for error recovery. Ignores
     * composed computations, returning the same promise.</p>
     *
     * @param error The error.
     */
    defer.reject = rejected;

    /**
     * Combine an array of promises into a promise that produces an array of
     * values.
     *
     * <p>Given an array of promises, produce a promise which (via a
     * reductive-bind) produces a new promise which is resolved when all other
     * promises are resolved.</p>
     */
    defer.all = function(promises) {
        // This would be so much cuter as
        //
        // promises.map(p => [ p ])
        //         .reduce((a, b) => a.then(a => b.then(b => a.concat(b))))
        //
        // But we can't have all the nice toys. Thank goodness [].concat has
        // been around since IE 5.5.
        if (promises.length) {
            return promises.map(function(p) {
                return p.then(function(v) { return [ v ]; });
            }).reduce(function(a, b) {
                return a.then(function(a) {
                    return b.then(function(b) {
                        return a.concat(b);
                    });
                });
            });
        }
        else {
            return resolved([]);
        }
    };

    /* Wrapper for dealing with the closure. */
    return defer;
});

/* global window, document, escape, XMLHttpRequest, ActiveXObject */
define('quant/loader',[ "quant/ready", "quant/promise" ], function(ready, Promise) {
    
    /**
     * A DOM-based service loader.
     *
     * <p>This object implements a number of mechanisms for reaching out to
     * third party services through DOM.  It deals with some of the minute
     * details, like ensuring unattached images aren't garbage collected before
     * they're fired, and providing a callbacks for loading events.</p>
     */
    function Loader(window, document) {
        var head;
        var counter  = 0;
        var arena = [];

        ready(function () {
            head = document.getElementsByTagName("head")[0];
        });

        /**
         * Prefetch and return an image object.
         */
        var prefetch = function(url) {
            var image = new Image();

            image.src = url;

            return image;
        };

        /**
         * Load an image.
         *
         * <p>Given a url, load an image using image prefetching and produce a
         * promise which resolves to that image, or is rejected on error.</p>
         *
         * @param url The URL to fetch the image from.
         *
         * @return A Promise[Image]
         */
        this.image = function(url) {
            return new Promise(function(resolve, reject) {
                var image = prefetch(url);

                arena.push(image);

                image.onload = function() {
                    resolve(image);

                    arena.shift();

                    delete image.onload;
                    delete image.onerror;
                };

                image.onerror = reject;
            });
        };

        /**
         * Fire and forget send beacon.
         *
         * <p>Wrapper for navigator.sendBeacon which defaults to using image
         * prefetching if navigator.sendBeacon is not present.</p>
         */
        this.beacon = function(url) {
            // TODO Perhaps pass `navigator` in?
            var navigator = window.navigator;

            if (navigator && navigator.sendBeacon) {
                navigator.sendBeacon(url);
            }
            else {
                prefetch(url);
            }
        };

        /**
         * Build a script tag.
         *
         * <p>Build a script tag providing onload and on error handlers.
         *
         * @param {String} src The source of the script tag.
         * @param {Number} nonDestructive When 0 tags are removed after loaded
         * @param {Function} success The handler to invoke when the script has
         * executed, <em>optional</em>.
         * @param {Function} error The handler to invoke if the script errors
         * instead of executing successfully.
         */
        var buildTag = function(src, resolve, reject) {
            var script = document.createElement("script");

            script.type = "text/javascript";
            script.src = src;

            /* This is maybe excessively defensive...
             * but just incase IE ever fixes the DOMContentLoaded event, we're
             * going to go ahead and have support for both models exist...and
             * keep a semaphore for invoking the callback so that if for any
             * reason we do get both events we degrade gracefully.
             */
            var dispatch = function() {
                resolve(script);

                script.onreadystatechange = null;
                script.onload = null;
                script.onerror = null;
            };

            /* Support DOMContentLoaded */
            script.onload = dispatch;

            /* Support IE... */
            script.onreadystatechange = function() {
                if (script.readyState in { "loaded": 1, "complete": 1 }) {
                    dispatch();
                }
            };

            script.onerror = reject;

            return script;
        };

        /**
         * Load a script, call a callback.
         *
         * @param {String} src The location of the script.
         * @return A promise which is resolved when the script is loaded, with
         * the script tag as a parameter.
         */
        this.script = function(src) {
            /* Insert at the top of the document.  We'd use the head but
             * document.head isn't supported by IE.
             */
            return new Promise(function(resolve, reject) {
                ready(function () {
                    var script = buildTag(src, resolve, reject);

                    // More IE8 quirks — insertBefore(e, <invalid node>)
                    // doesn't work also hasChildNodes() doesn't work.
                    if (head.firstChild) {
                        head.insertBefore(script, head.firstChild);
                    } else {
                        head.appendChild(script);
                    }
                });
            });
        };
    }

    return Loader;
});

/*jshint evil: true */
/* global window */

define('quant/json',[],function () {
    'use strict';
    /**
     * A super tiny JSON.stringify.  Doesn't validate, and isn't necessarily safe.
     */

    var localJSON = window.JSON || {};

    if (typeof localJSON.stringify === 'undefined' ||
            localJSON.stringify({'test': ['1']}) !== '{"test":["1"]}') {
        /**
         * A simple, light, pure javascript implementation of JSON.stringify
         *
         * @param object The object to turn into a JSON string.
         * @return The json string.
         */
        localJSON.stringify = function (object) {
            var type = typeof(object);
            if (type !== 'object' || object === null) {
                // simple data type
                if (type === 'string') {
                    object = '"' + object + '"';
                }
                return String(object);
            }
            else {
                // recurse array or object
                var name, value, json = [],
                array = (object && object.constructor === Array);

                for (name in object) {
                    value = object[name];
                    type = typeof(value);

                    /* XXX 2012-07-11T16:18:42Z-0700
                     * Skip functions, this addresses a bug in IE where if a
                     * collection has a prototype added it gets included when
                     * coerced into a string.
                     */
                    if (type === 'function') {
                        continue;
                    }
                    if (type === 'string') {
                        value = '"' + value + '"';
                    }
                    else if (type === 'object' && value !== null) {
                        value = localJSON.stringify(value);
                    }
                    json.push((array ? '' : '"' + name + '":') + String(value));
                }

                return (array ? '[' : '{') + String(json) + (array ? ']' : '}');
            }
        };

        /**
         * A simple, light JSON parser.
         * <p><b>Note</b> this uses the incredibly unsafe eval() method.</p>
         * @param string The JSON body.
         * @return An object.
         */
        localJSON.parse = localJSON.parse || function (string) {
            return eval('(' + string + ')');
        };
    }

    return localJSON;
});

/* global jQuery */
define('quant/event',[],function () {
    
    /**
     * Access event APIs.
     *
     * <p>This object provides a set of utility functions for accessing event
     * APIs on a fairly portable fashion, adjusted for the use-case centric for
     * QuantJS.  This means that there's a bit of disparity between the way
     * events are attached and the way they are dispatched, as this is designed
     * to be portable mechanism for doing both within QuantJS without depending
     * on external libraries.</p>
     *
     * <p>Events using this interface are dispatched using the DOM Level 3 API,
     * but falling back to using jQuery if it is available.  This is so
     * partners who whish to use the DOM Event API to observe delivery state
     * changes can support IE 8 by utilizing jQuery if desired.</p>
     *
     * <p>Listening to events using this interface relies on the DOM Level 3
     * API, and falls back to using jQuery if it is present, but then further
     * falls back to using the MSIE Event API of <code>attachEvent</code> and
     * <code>removeEvent</code>.</p>
     */
    function Event() {
        /**
         * Listen for an event of a given type.
         *
         * <p>Given a node, an event type, and a callback function add the
         * given function as an event listener for all events originating from
         * the given node or one of its descendants, using whichever method
         * seems most appropriate.</p>
         *
         * @param node {Node} The DOM node.
         * @param type {String} The event type.
         * @param callback {Function} The callback to invoke.
         */
        this.add = function (node, type, callback) {
            if (node.addEventListener) {
                node.addEventListener(type, callback);
            }
            else if (typeof jQuery === "function") {
                jQuery(node).on(type, callback);
            }
            else if (node.attachEvent) {
                node.attachEvent("on" + type, callback);
            }
        };

        /**
         * Remove event listener.
         *
         * <p>Remove a given event listener using the appropriate method.</p>
         *
         * @param node {Node} The DOM node.
         * @param type {String} The event type.
         * @param callback {Function} The callback to remove.
         */
        this.remove = function (node, type, callback) {
            if (node.removeEventListener) {
                node.removeEventListener(type, callback);
            }
            else if (typeof jQuery === "function") {
                jQuery(node).off(type, callback);
            }
            else if (node.detachEvent) {
                node.detachEvent("on" + type, callback);
            }
        };

        /**
         * Dispatch an event on a given node.
         *
         * <p>Given a node, an event type, and a set of properties dispatch an
         * event using DOM Level 3 if possible, falling back to jQuery if
         * necessary.</p>
         *
         * @param node {Node} The DOM Node.
         * @param type {String} The event type.
         * @param properties {Object} Any additional properties (optional)
         */
        this.trigger = function (node, type, properties) {
            var doc = node.ownerDocument;

            if (node.dispatchEvent && doc.createEvent) {
                var nodeEvent = doc.createEvent("Event");

                nodeEvent.initEvent(type, true, true);

                if (typeof properties !== "undefined") {
                    for (var key in properties) {
                        // Avoid overwriting any pre-existing properties of the
                        // nodeEvent object
                        if (!(key in nodeEvent)) {
                            nodeEvent[key] = properties[key];
                        }
                    }
                }

                node.dispatchEvent(nodeEvent);
            }
            else if (typeof jQuery === "function") {
                jQuery(node).trigger(type, properties);
            }
        };
    }

    return new Event();
});

/* global window */
define('quant/consent/truste',[ "quant/json", "quant/promise", "quant/event", "quant/now" ],
function (Json, Promise, event, now) {
    return function TrustEConsentProvider(windows, window, top,
    PrivacyManagerAPI, authority, consentType, consentTarget, domain)
    {
        var required = false;
        var sendCommand;
        var parameters = {};

        var serialize = function(settings) {
            // So...we decided a long time ago that "pde" meant "denied"
            // "expressed", and "pai" meant "approved", "implied" yet the API
            // doesn't return "expressed" it returns either "implied" or
            // "asserted"
            var source = settings.source[0];

            return "p" + settings.consent[0] + (source == "a" ? "e" : "i");
        };

        if (typeof PrivacyManagerAPI === "object" &&
            typeof PrivacyManagerAPI.callApi === "function")
        {
            required = true;

            // sendCommand is a little dubious, this actually just calls a
            // global function.
            sendCommand = function(parameters, command, consentType, consentTarget) {
                // It appears that this thing never returns false, but there's
                // code that assumes it does
                var settings = PrivacyManagerAPI.callApi(
                    command, consentTarget, domain, authority, consentType);

                parameters["cm"] = serialize(settings);

                return Promise.resolve(true);
            };
        }
        else {
            sendCommand = function(parameters, command, consentType, consentTarget) {
                if (windows.depth > 0) {
                    // NOTE This is mostly just to figure out whether or not this
                    // ever happens. For the purposes of ensuring this integration
                    // can be removed. It appears that the first implementation in
                    // quant.js had a very silly bug in which it would assume if it
                    // got a "PrivacyManagerAPI" response *at all* it would mean
                    // there was consent.
                    event.add(window, "message", function(evt) {
                        var msg = evt.data;

                        if (typeof msg === "string" && msg.indexOf("PrivacyManagerAPI") > 0) {
                            try {
                                msg = Json.parse(msg);
                            }
                            catch (error) {
                                return;
                            }
                        }
                        else if (typeof msg.PrivacyManagerAPI !== "undefined") {
                            var settings = msg.PrivacyManagerAPI;

                            // These come in as stragglers due to our rules, so
                            // just let them get tacked into the request and deal
                            // with them in lighttpd if that's how this went down.
                            parameters["cm"] = serialize(settings);
                        }
                    });

                    top.postMessage(Json.stringify({
                        PrivacyManagerAPI: {
                            timestamp: now(),
                            action: command,
                            self: consentTarget,
                            domain: domain,
                            authority: authority,
                            type: consentType
                        }
                    }), '*');
                }

                return Promise.resolve(true);
            };
        }

        /**
         * Query TRUSTe/TrustArc compatible privace manager APIs for consent.
         *
         * <p>Produces a promise which always allows the pixel and is always
         * resolved, but *may* update the parameters after this promise is
         * resolved.</p>
         */
        this.consent = function(parameters) {
            return sendCommand(parameters, "getConsent", consentType,
                consentTarget);
        };

        this.parameters = parameters;
    };
});

define('quant/consent/uspapi',[ "quant/promise", "quant/json", "quant/event", "quant/now" ],
function(Promise, Json, event, now) {
    var USP_API_VERSION = 1;

    // Resolver names
    var RESOLVE = 0;
    var REJECT = 1;

    return function USPAPIConsentProvider(windows, window, log, frameName) {
        var sendCommand;

        if (typeof window.__uspapi === "function") {
            sendCommand = function(command, version) {
                return new Promise(function(resolve, reject) {
                    window.__uspapi("getUSPData", version, function(response, success) {
                        if (response && typeof response.uspString === "string") {
                            resolve(response);
                        }
                        else {
                            reject(response);
                        }
                    });
                }).catch(function(error) {
                    log.error("[USPAPI] unsuccessful: ", error);

                    return true;
                });
            };
        }
        else {
            var uspFrame = windows.locate(frameName);
            var resolvers = {};

            event.add(window, "message", function(evt) {
                var message = evt.data;

                if (typeof message === "string" && message[0] == '{') {
                    try {
                        message = Json.parse(message);
                    }
                    catch (error) {
                        return;
                    }
                }

                if (message.hasOwnProperty("__uspapiReturn")) {
                    var ret = message.__uspapiReturn;
                    var callId = ret.callId;

                    var resolver = resolvers[callId];

                    if (typeof resolver === "undefined") {
                        return;
                    }

                    if (ret.success) {
                        resolver[RESOLVE](ret.returnValue);
                    }
                    else {
                        resolver[REJECT](ret.returnValue);
                    }
                }
            });

            sendCommand = function(command, version) {
                var uspFrame = windows.locate(frameName);

                if (typeof uspFrame === "undefined") {
                    return Promise.resolve(void(0));
                }

                var callId = now();

                return new Promise(function(resolve, reject) {
                    resolvers[callId] = [ resolve, reject ];

                    uspFrame.postMessage({
                        __uspapiCall: {
                            command: command,
                            version: version,
                            callId: callId
                        }
                    });
                });
            };
        }

        this.consent = function(parameters) {
            return sendCommand("getUSPData", USP_API_VERSION).then(function(response) {
                if (response && typeof response.uspString === "string") {
                    parameters["us_privacy"] = response.uspString;
                }

                return true;
            });
        };
    };
});

define('quant/consent-manager',[ "quant/promise", "quant/json" ], function(Promise, Json) {
    /**
     * @constructor
     * The global consent provider.
     *
     * <p>This is a basic gate-keeper for determining when and where data
     * should be collected. There are a few basic types of consent, here.
     * Must be used for all actions which share data.
     *
     * pixel.send(pcode, params) for instance must call it.
     *
     * @param providers An array of consent provider objects.
     */
    return function ConsentManager (providers) {
        var parameters = {};
        var promise;

        var consent = function(cb) {
            // I have written this code *twice*, if I write it a third time I
            // will create a `once()` function or similar.
            if (typeof promise === "undefined") {

                promise = Promise.all(providers.map(function(provider) {
                    return provider.consent(parameters);
                })).then(function(signals) {
                    // All consent providers need to decide that they can move
                    // forward. (Many of them default to "if I don't see code
                    // on the page, yes").
                    return signals.reduce(function(a, b) { return a && b; }, true);
                });
            }

            return promise.then(function(allowed) {
                if (allowed) {
                    return cb();
                }
            });
        };

        /**
         * Execute an action given consent.
         *
         * <p>Given consent has been granted from the user if necessary,
         * execute the given action.</p>
         *
         * @param cb A function to execute.
         * @return A promise producing the result.
         */
        this.consent = consent;

        /**
         * Wrap a function as a consented action.
         *
         * <p>Using function currying and such, generate a passthrough function
         * which produces a promise of the result that the supplied function
         * returns.</p>
         *
         * <p>If the supplied function produces a promise, the produced
         * function naturally binds that promise.</p>
         *
         * <p><code>this</code> and <code>arguments</code> are passed
         * through.</p>
         *
         * @param wrapped The function to wrap.
         * @returns A function which produces a promise of the given result.
         */
        this.wrap = function(wrapped) {
            return function() {
                var self = this;
                var args = arguments;

                return consent(function() {
                    return wrapped.apply(self, args);
                });
            };
        };

        /**
         * The parameters to add to pixel.
         *
         * <p>A read-only copy of the parameters to be merged into pixel when
         * the pixel is sent.</p>
         *
         * <p>This should be used as a starting point for all pixel path
         * parameters.</p>
         *
         * @return An object of the path parameters and their values.
         */
        this.parameters = parameters;
    };
});

define('quant/consent/tcf2.0',[ "quant/promise", "quant/json", "quant/event", "quant/now" ],
function(Promise, Json, event, now) {
    var LOG_PREFIX = "[TCF2]: ";

    // Resolver names
    var RESOLVE = 0;
    var REJECT = 1;

    var PURPOSE_DATA_COLLECT = "1";

    var QC_TCF_VENDOR_ID = 11;
    var QC_TCF_REQUIRED_PURPOSES = [PURPOSE_DATA_COLLECT, "3", "7", "8", "9", "10"];
    var QC_TCF_CONSENT_FIRST_PURPOSES = [PURPOSE_DATA_COLLECT, "3"];
    var QC_TCF_CONSENT_ONLY_PUPROSES = [PURPOSE_DATA_COLLECT, "3"];

    var CMP_VERSION = 2;
    var CMP_COMMAND_ADD_EVENT_LISTENER = "addEventListener";
    var CMP_RETURN_KEY = "__tcfapiReturn";
    var CMP_CALL_KEY = "__tcfapiCall";

    /**
     * Resolve consent from various input signals.
     *
     * Note that we do not validate CMP ID when resolving consent. This is left
     * to lighttpd. Validating the CMP ID in quant.js would either require
     * maintaining a hard-coded list of registered valid CMP IDs (gross), or
     * making yet another HTTP request to fetch a list of validated CMP IDs
     * (extremely undesirable for performance reasons).
     *
     * @return true if consented, false if consent denied
     */
    function resolveConsent(requiredPurposes, tcData) {
        var gdprApplies = tcData.gdprApplies;
        var purposes = tcData.purpose;
        var vendors = tcData.vendor;
        var qcConsent = vendors && vendors.consents && vendors.consents[QC_TCF_VENDOR_ID];
        var qcInterest = vendors && vendors.legitimateInterests && vendors.legitimateInterests[QC_TCF_VENDOR_ID];
        var restrictions = tcData.publisher ? tcData.publisher.restrictions : {};

        if (!gdprApplies) {
            return true;
        }

        return requiredPurposes.map(function(purpose) {
            var purposeConsent = purposes.consents ? purposes.consents[purpose] : false;
            var purposeInterest = purposes.legitimateInterests ? purposes.legitimateInterests[purpose] : false;

            var qcRestriction = restrictions && restrictions[purpose]
                ? restrictions[purpose][QC_TCF_VENDOR_ID]
                : null;

            if (qcRestriction === 0) {
                // publisher has flatly disallowed purpose for Quantcast :-(
                return false;
            }

            // Seek consent or legitimate interest based on our default legal
            // basis for the purpose, falling back to the other if possible.

            if (
                // we have positive vendor consent
                qcConsent &&
                // there is positive purpose consent
                purposeConsent &&
                // publisher does not require legitimate interest
                qcRestriction !== 2 &&
                // purpose is a consent-first purpose or publisher has explicitly restricted to consent
                (QC_TCF_CONSENT_FIRST_PURPOSES.indexOf(purpose) != -1 || qcRestriction === 1)
            ) {
                return true;

            } else if (
                // publisher does not require consent
                qcRestriction !== 1 &&
                // we have legitimate interest for vendor
                qcInterest &&
                // there is legitimate interest for purpose
                purposeInterest &&
                // purpose's legal basis does not require consent
                QC_TCF_CONSENT_ONLY_PUPROSES.indexOf(purpose) == -1 &&
                // purpose is a legitimate-interest-first purpose or publisher has explicitly restricted to legitimate interest
                (QC_TCF_CONSENT_FIRST_PURPOSES.indexOf(purpose) == -1 || qcRestriction === 2)
            ) {
                return true;
            }

            // no legal basis :-(
            return false;
        }).reduce(function(a, b) {
            return a && b;
        }, true);
    }

    function TCF2ConsentProvider(windows, window, log, frameName) {
        var promise;
        var sendCommand;

        if (typeof window.__tcfapi === "function") {
            sendCommand = function(command, parameter) {
                return new Promise(function(resolve, reject) {
                    window.__tcfapi(command, CMP_VERSION, function(result, success) {
                        if (success) {
                            var eventStatus = result.eventStatus;
                            if (
                                // only resolve `addEventListener` if GDPR doesn't apply, or the user has signaled consent
                                command !== CMP_COMMAND_ADD_EVENT_LISTENER
                                || (!result.gdprApplies || eventStatus === "useractioncomplete" || eventStatus === "tcloaded")
                            ) {
                                resolve(result);
                            }
                        }
                        else {
                            reject(result);
                        }
                    }, parameter);
                });
            };
        } else {
            var resolvers = {};
            var commands = {};

            event.add(window, "message", function(evt) {
                // Only decode it if we need to, newer browsers *do* support
                // non-string messages (but IE8 *doesnt*)
                var message = evt.data;

                if (typeof message === "undefined") {
                    log.error(LOG_PREFIX + "Recieved undefined message");
                    return;
                }

                if (typeof message === "string" && message[0] == "{") {
                    try {
                        message = Json.parse(message);
                    }
                    catch (error) {
                        return;
                    }
                }

                if (message.hasOwnProperty(CMP_RETURN_KEY)) {
                    var ret = message[CMP_RETURN_KEY];
                    var callId = ret.callId;

                    var resolver = resolvers[callId];

                    if (typeof resolver === "undefined") {
                        return;
                    }

                    var result = ret.returnValue;

                    if (ret.success) {
                        if (
                            // only resolve `addEventListener` if GDPR doesn't apply, or the user has signaled consent
                            commands[callId] !== CMP_COMMAND_ADD_EVENT_LISTENER
                            || (!result.gdprApplies || result.eventStatus === "useractioncomplete" || result.eventStatus === "tcloaded")
                        ) {
                            resolver[RESOLVE](result);
                        }
                    }
                    else {
                        // Hopefully returnValue has something useful in it,
                        // there's no guarantees.
                        resolver[REJECT](result);
                    }
                }
            });

            sendCommand = function(command, parameter) {
                // Try to find the frame every time, assuming it's now here..
                var cmpFrame = windows.locate(frameName);

                if (typeof cmpFrame === "undefined") {
                    // No locator present, no CMP, assume consent.
                    return Promise.resolve({ gdprApplies: false });
                }

                var callId = now();

                return new Promise(function(resolve, reject) {
                    resolvers[callId] = [ resolve, reject ];
                    commands[callId] = command;

                    var payload = {};
                    payload[CMP_CALL_KEY] = {
                        command: command,
                        parameter: parameter,
                        version: CMP_VERSION,
                        callId: callId
                    };

                    cmpFrame.postMessage(Json.stringify(payload), "*");
                });
            };
        }

        this.consent = function(parameters) {
            if (typeof promise === "undefined") {
                promise = sendCommand(CMP_COMMAND_ADD_EVENT_LISTENER).then(function(tcData) {
                    var applies = tcData.gdprApplies && tcData.gdprApplies != "false";

                    if (applies) {
                        parameters["gdpr"] = 1;

                        // Collect the consent string so it can be shared with
                        // the pixel service if pixels are sent.
                        parameters["gdpr_consent"] = tcData.tcString;
                    } else {
                        // Avoid overwriting "gdpr" parameter which might have
                        // been set by another GDPR consent provider.
                        parameters["gdpr"] = parameters["gdpr"] || 0;
                    }

                    return resolveConsent(QC_TCF_REQUIRED_PURPOSES, tcData);
                }).catch(function (error) {

                    log.error("[TCF2.0] unsuccessful: ", error);

                    // Assume if the CMP did not follow the protocol, that we
                    // have consent and allow fallback to GeoIP on Lighttpd to
                    // filter out European-Originating messages without a
                    // consent signal, but avoid overwriting "gdpr" parameter
                    // which might have been set by another GDPR consent
                    // provider.
                    parameters["gdpr"] = parameters["gdpr"] || 0;

                    return true;
                });
            }

            return promise;
        };
    }

    // exposed for testing
    TCF2ConsentProvider.resolveConsent = resolveConsent;

    return TCF2ConsentProvider;
});

define('quant/qtrack',[], function () {
  var ALLOWED_EVENTS = [
    'PageView', 'ViewContent', 'Search', 'AddToWishlist',
    'AddToCart', 'InitiateCheckout', 'AddPaymentInfo',
    'Purchase', 'Lead', 'Register', 'StartTrial',
    'Subscribe', 'SubmitApplication'
  ];
  var INIT = 'init';
  var TRACK = 'track';
  var TRACK_CUSTOM = 'trackCustom';
  var PROP_REPLACEMENTS = {
    'order_id': 'orderid',
    'value': 'revenue'
  }

  // pCodes initialized on the page
  var pCodes;
  // Properties to be added to every pixel fire
  var extraProps;

  function assignObject(to, from) {
    for (var key in from) if (from.hasOwnProperty(key)) {
      to[key] = from[key];
    }
  }

  /**
   * Triggers a pixel fire using the underlying _qevents API
   */
  function triggerPixel(event, props, isCustom) {
    for (var i = 0; i < pCodes.length; i++) {
      var input = {
        qacct: pCodes[i],
        labels: isCustom ? event : '_fp.event.' + event,
        event: 'refresh'
      };
      assignObject(input, extraProps);
      if (props !== undefined && props !== null) {
        for (var key in props) if (props.hasOwnProperty(key)) {
          if (key === 'product_id' && props[key].constructor === Array) {
            props[key] = props[key].join(',');
          }
          input[PROP_REPLACEMENTS[key] || key] = props[key];
        }
      }
      window._qevents.push(input);
    }
  }

  /**
   * qtrack API
   */
  function _qtrack(action, event, props) {
    if (action === INIT) {
      if (pCodes.indexOf(event) !== -1) return;
      pCodes.push(event);
      var input = { qacct: event };
      assignObject(extraProps, props);
      assignObject(input, extraProps);
      window._qevents.push(input);
    } else if (action === TRACK) {
      if (ALLOWED_EVENTS.indexOf(event) !== -1) {
        triggerPixel(event, props, false);
      } else {
        // eslint-disable-next-line no-console
        console.warn('Unsupported event by track, please use ' + TRACK_CUSTOM + ' for this event.');
      }
    } else if (action === TRACK_CUSTOM) {
      triggerPixel(event, props, true);
    }
  }

  /**
   * Initializes the qtrack function on the window and set's the default values.
   */
  return function init() {
    if (!window.qtrack) {
      window.qtrack = function(){
        window.qtrack.impl.apply(window.qtrack, arguments);
      };
    }
    // if already initialized
    if (window.qtrack.impl) return;

    pCodes = [];
    extraProps = {};

    window.qtrack.impl = _qtrack;
    if (window.qtrack && window.qtrack.q) {
      while (window.qtrack.q.length > 0) {
        _qtrack.apply(_qtrack, window.qtrack.q.shift());
      }
    }
  };
});

define('quant/hashing',[],function() {
    function Hashes() {

        /**
         *
         * Implementation of Fowler–Noll–Vo hash function
         *
         * @param string
         *
         * @return string hash value
         **/
        this.FNV = function(s) {

            var h1, h2, hash1, hash2;

            h1 = 0x811c9dc5;

            h2 = 0xc9dc5118;
            hash1 = doFNVHash(h1, s);
            hash2 = doFNVHash(h2, s);

            return (Math.round(Math.abs(hash1 * hash2) / 65536)).toString(16);
        };


        var doFNVHash = function(h, s) {
            var i;

            for (i = 0; i < s.length; i++) {
                h ^= s.charCodeAt(i);
                h += (h << 1) + (h << 4) + (h << 7) + (h << 8) + (h << 24);
            }

            return h;
        };


        /** 
         *   
         * www.movable-type.co.uk/scripts/sha256.html                                                     
         * 
         * Generates SHA-256 hash of string.
         *
         * @param   {string} msg - (Unicode) string to be hashed.
         * @returns {string} Hash of msg as hex character string.
         *
         */
        this.SHA256 = function(msg) {

            // note use throughout this routine of 'n >>> 0' to coerce Number 'n' to unsigned 32-bit integer

            msg = utf8Encode(msg);

            // varants [§4.2.2]
            var K = [
                0x428a2f98, 0x71374491, 0xb5c0fbcf, 0xe9b5dba5, 0x3956c25b, 0x59f111f1, 0x923f82a4, 0xab1c5ed5,
                0xd807aa98, 0x12835b01, 0x243185be, 0x550c7dc3, 0x72be5d74, 0x80deb1fe, 0x9bdc06a7, 0xc19bf174,
                0xe49b69c1, 0xefbe4786, 0x0fc19dc6, 0x240ca1cc, 0x2de92c6f, 0x4a7484aa, 0x5cb0a9dc, 0x76f988da,
                0x983e5152, 0xa831c66d, 0xb00327c8, 0xbf597fc7, 0xc6e00bf3, 0xd5a79147, 0x06ca6351, 0x14292967,
                0x27b70a85, 0x2e1b2138, 0x4d2c6dfc, 0x53380d13, 0x650a7354, 0x766a0abb, 0x81c2c92e, 0x92722c85,
                0xa2bfe8a1, 0xa81a664b, 0xc24b8b70, 0xc76c51a3, 0xd192e819, 0xd6990624, 0xf40e3585, 0x106aa070,
                0x19a4c116, 0x1e376c08, 0x2748774c, 0x34b0bcb5, 0x391c0cb3, 0x4ed8aa4a, 0x5b9cca4f, 0x682e6ff3,
                0x748f82ee, 0x78a5636f, 0x84c87814, 0x8cc70208, 0x90befffa, 0xa4506ceb, 0xbef9a3f7, 0xc67178f2 ];

            // initial hash value [§5.3.3]
            var H = [
                0x6a09e667, 0xbb67ae85, 0x3c6ef372, 0xa54ff53a, 0x510e527f, 0x9b05688c, 0x1f83d9ab, 0x5be0cd19 ];

            // PREPROCESSING [§6.2.1]

            msg += String.fromCharCode(0x80);  // add trailing '1' bit (+ 0's padding) to string [§5.1.1]

            // convert string msg into 512-bit blocks (array of 16 32-bit integers) [§5.2.1]
            var l = msg.length/4 + 2; // length (in 32-bit integers) of msg + ‘1’ + appended length
            var N = Math.ceil(l/16);  // number of 16-integer (512-bit) blocks required to hold 'l' ints
            var M = new Array(N);     // message M is N×16 array of 32-bit integers

            for (var i=0; i<N; i++) {
                M[i] = new Array(16);
                for (var j=0; j<16; j++) { // encode 4 chars per integer (64 per block), big-endian encoding
                    M[i][j] = (msg.charCodeAt(i*64+j*4+0)<<24) | (msg.charCodeAt(i*64+j*4+1)<<16)
                            | (msg.charCodeAt(i*64+j*4+2)<< 8) | (msg.charCodeAt(i*64+j*4+3)<< 0);
                } // note running off the end of msg is ok 'cos bitwise ops on NaN return 0
            }
            // add length (in bits) into final pair of 32-bit integers (big-endian) [§5.1.1]
            // note: most significant word would be (len-1)*8 >>> 32, but since JS converts
            // bitwise-op args to 32 bits, we need to simulate this by arithmetic operators
            var lenHi = ((msg.length-1)*8) / Math.pow(2, 32);
            var lenLo = ((msg.length-1)*8) >>> 0;
            M[N-1][14] = Math.floor(lenHi);
            M[N-1][15] = lenLo;


            // HASH COMPUTATION [§6.2.2]

            for (i=0; i<N; i++) {
                var W = new Array(64);

                // 1 - prepare message schedule 'W'
                for (var t=0;  t<16; t++) W[t] = M[i][t];
                for (t=16; t<64; t++) {
                    W[t] = (delta1(W[t-2]) + W[t-7] + delta0(W[t-15]) + W[t-16]) >>> 0;
                }

                // 2 - initialise working variables a, b, c, d, e, f, g, h with previous hash value
                var a = H[0], b = H[1], c = H[2], d = H[3], e = H[4], f = H[5], g = H[6], h = H[7];

                // 3 - main loop (note '>>> 0' for 'addition modulo 2^32')
                for (t=0; t<64; t++) {
                    var T1 = h + sigma1(e) + Ch(e, f, g) + K[t] + W[t];
                    var T2 =     sigma0(a) + Maj(a, b, c);
                    h = g;
                    g = f;
                    f = e;
                    e = (d + T1) >>> 0;
                    d = c;
                    c = b;
                    b = a;
                    a = (T1 + T2) >>> 0;
                }

                // 4 - compute the new intermediate hash value (note '>>> 0' for 'addition modulo 2^32')
                H[0] = (H[0]+a) >>> 0;
                H[1] = (H[1]+b) >>> 0;
                H[2] = (H[2]+c) >>> 0;
                H[3] = (H[3]+d) >>> 0;
                H[4] = (H[4]+e) >>> 0;
                H[5] = (H[5]+f) >>> 0;
                H[6] = (H[6]+g) >>> 0;
                H[7] = (H[7]+h) >>> 0;
            }

            // convert H0..H7 to hex strings (with leading zeros)
            for (h=0; h<H.length; h++) H[h] = ('00000000'+H[h].toString(16)).slice(-8);

            // concatenate H0..H7, with separator if required
            var separator = '';

            return H.join(separator);

        };

        function utf8Encode(str) {
            return unescape(encodeURIComponent(str));
        }

        /**
         * Rotates right (circular right shift) value x by n positions [§3.2.4].
         */
        function ROTR(n, x) {
            return (x >>> n) | (x << (32-n));
        }

        /**
         * Logical functions [§4.1.2].
         */
        function sigma0(x) { return ROTR(2,  x) ^ ROTR(13, x) ^ ROTR(22, x); }
        function sigma1(x) { return ROTR(6,  x) ^ ROTR(11, x) ^ ROTR(25, x); }
        function delta0(x) { return ROTR(7,  x) ^ ROTR(18, x) ^ (x>>>3);  }
        function delta1(x) { return ROTR(17, x) ^ ROTR(19, x) ^ (x>>>10); }
        function Ch(x, y, z)  { return (x & y) ^ (~x & z); }          // 'choice'
        function Maj(x, y, z) { return (x & y) ^ (x & z) ^ (y & z); } // 'majority'
    }
    return new Hashes();
});

define('quant/normalize',["quant/hashing"], function(hashing) {

    var createConfigOptions = function (suffix, mayBeObject, defaults, pCode, ruleFetchOutcome) {
        var opt = {},
            uh = null,
            USERHASH_PATTERN_EMAIL = /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
            USERHASH_PATTERN_SHA256 = /^[A-Fa-f0-9]{64}$/,
            USERHASH_TYPE_EMAIL = 0,
            USERHASH_TYPE_SHA256 = 1,
            USERHASH_TYPE_UNKNOWN = 2,
            uht = USERHASH_TYPE_UNKNOWN,
            fieldsConsumed = {},
            k;

        for (k in mayBeObject) {
            fieldsConsumed[k] = mayBeObject[k] !== undefined;
            if (!mayBeObject.hasOwnProperty(k) || typeof (mayBeObject[k]) != 'string') {
                continue;
            }
            if (k === 'uid' || k === 'uh') {
                /*
                * We allow the publisher to give us their own UID data in the 'uid'
                * property of _qoptions, but we don't actually want to know their
                * actual mapping, so we feed it to qhash before sending it
                * along to lighttpd. We check if the uid follows a pattern before we hash it to
                * give us additional information about the uid without actually looking at the
                * actual value.
                * Internally, we still call it 'uh' so we also rename the entry.
                */
                if (mayBeObject[k].match(USERHASH_PATTERN_SHA256)) {
                    uht = USERHASH_TYPE_SHA256;
                    uh = mayBeObject[k].toLowerCase();
                } else {
                    if (mayBeObject[k].match(USERHASH_PATTERN_EMAIL)) {
                        uht = USERHASH_TYPE_EMAIL;
                        mayBeObject[k] = mayBeObject[k].toLowerCase();
                    }
                    if (mayBeObject[k] !== '') {
                        uh = hashing.SHA256(mayBeObject[k]);
                    }
                }
                delete mayBeObject[k];
                continue;
            }
            if (k === 'qacct') {
                continue;
            }
            // Only add the field if there is data to add to the pixel
            if (mayBeObject[k].length > 0) {
                opt[k + suffix] = encodeURIComponent(mayBeObject[k]);
            } else {
                fieldsConsumed[k] = false;
            }
        }

        for (k in defaults) if (defaults.hasOwnProperty(k) && typeof (defaults[k]) == 'string' && !fieldsConsumed[k]) {
            opt[k + suffix] = encodeURIComponent(defaults[k]);
        }

        opt['rf' + suffix] = '' + ruleFetchOutcome;

        if (typeof uh === 'string') {
            mayBeObject['uh'] = uh;
            opt['uh' + suffix] = encodeURIComponent(uh);
        }

        opt['uht' + suffix] = "" + uht;
        opt['a' + suffix] = pCode;
        return opt;
    };

    return createConfigOptions;
});
/* global window _qacct _qoptions _qevents __cmp __uspapi */
define('quant.js',[ "quant/origin", "quant/windows", "quant/log", "quant/loader",
    "quant/consent/truste", "quant/consent/uspapi", 
    "quant/consent-manager", "quant/consent/tcf2.0", "quant/qtrack", "quant/normalize", "quant/hashing" ],
function(origin, Windows, Logger, Loader, TrustEConsentProvider,
    USPAPIConsentProvider, ConsentManager,
    TCF2ConsentProvider, qtrack, normalize, hashing) {

    if (typeof window.__qc === "undefined") {
        // TODO Remove this closure it is no longer necessary.
        window.__qc = (function(win, doc, nav) {
            if (win.__qc) {
                return win.__qc;
            }

            // Domains
            var DOMAIN_QSERVE = "quantserve.com",
                DOMAIN_QCOUNT = "quantcount.com",
                DOMAIN_TRUSTE = "truste.com";

            // Locator iframe names.
            var USP_IFRAME_NAME = "__uspapiLocator",
                CMP_IFRAME_NAME = "__cmpLocator",
                TCF2_CMP_IFRAME_NAME = "__tcfapiLocator";

            var domain = origin(doc);
            var windows = new Windows(win, win.top);

            var loader = new Loader(win, doc);
            var log = new Logger(loader, DOMAIN_QCOUNT);

            var consentManager = new ConsentManager([
                new TrustEConsentProvider(windows, win, win.top,
                    win.PrivacyManagerAPI, DOMAIN_TRUSTE, "advertising",
                    DOMAIN_QSERVE, domain),
                new USPAPIConsentProvider(windows, win, log, USP_IFRAME_NAME),
                new TCF2ConsentProvider(windows, win, log, TCF2_CMP_IFRAME_NAME)
            ]);

            // If you are new to this codebase, the entry point is the api('init') function
            var RESERVED_FIELDS = ['a', 'ce', 'cm', 'dst', 'enc', 'fpa', 'fpan', 'je', 'ns', 'ogl', 'rf', 'tzo', 'sr'],

                /**
                 * Cookie expiry time interval
                 * We changed it because it broke tests when day "after 13 months" was PST and test were being run in PDT.
                 * Intention here is to have cookie expiry before 13 months and not at 13th month
                 */
                COOKIE_EXP_TIME = 33868800000, // (13 months - 2 days), in milliseconds

                // Constants for results of rule fetch
                RULE_FETCH_OK = 0,
                RULE_FETCH_FAILED = 1,
                RULE_FETCH_INDETERMINATE = 2,
                RULE_FETCH_UNKNOWN = 3,
                RULE_FETCH_UNSUPPORTED = 4,
                RULE_FETCH_EMPTY = 6,

                // Version string will be interpolated by `make` command
                QUANT_JS_VERSION = '40d1d9f5-20220725143430';

            var SCRIPT_LOADER_MODERN = 1,
                SCRIPT_LOADER_LEGACY = 2,
                SCRIPT_LOADER_UNSUPPORTED = 3,
                SCRIPT_LOADER;

            // Grab references to global objects in order to protect us
            // from later malicious overwrites.
            var SD = [
                    '4dcfa7079941', //adnxs.com
                    '127fdf7967f31', // appnexus.com
                    '588ab9292a3f', // invitemedia.com
                    '32f92b0727e5', // 247realmedia.com
                    '22f9aa38dfd3', // cpmadvisors.com
                    'a4abfe8f3e04', // bluelithium.com
                    '18b66bc1325c', // googleadservices.com
                    '958e70ea2f28', // doubleclick.net
                    'bdbf0cb4bbb', // atdmt.com
                    '65118a0d557', // yieldmanager.com
                    '40a1d9db1864', // fimserve.com
                    '18ae3d985046', // tribalfusion.com
                    '3b26460f55d' // trafficmp.com
                ],
                RULE_EVENT_TYPE = 'rule',
                ALLOW_MULTI_PIXEL = true,

                MEDIA_TYPE_WEBPAGE = 'webpage',
                MEDIA_TYPE_AD = 'ad',

                EVENT_TYPE_LOAD = 'load',

                /**
                 * Has quantserve() ever been called?
                **/
                isInitialized = false,

                isQacctConsumed = false,

                /**
                 * We also embed the quantjs tag in ads
                 * For ads, we set media=ad:
                 *    _qoptions = {qacct: 'p-cde123', media: 'ad'};
                 *
                 * This tells quantjs to correctly set media type in pixel call
                 * @type number
                 **/
                isAd = 0,

                /**
                 * Queue containing all the serialized params for pixels waiting to be fired
                 * @type Array<Object>
                 **/
                pixelQueue = [],

                /**
                 * A list of rules where each rule is of the shape:
                 * @type Rule:
                 *    {
                 *      p: string, // The p-code
                 *      r: () => any, // Reducer function
                 *      a: () => object, // Action function
                 *      f: Array<bool>, // Flags indicating which conditions have been fulfilled
                 *      c: Array<Array<any>>
                 *   }
                 *
                **/
                rules = [],

                /**
                 * A list of p-codes for which rules have been loaded into th system.
                 * These are ignored when fetching rules.
                **/
                pCodesWithRules = [],

                /**
                 * An array containing all p-codes encountered so far
                 * @type Array<string>
                **/
                pcodesFound = [],

                /**
                 * A map containing outcomes of rule fetches for each pcode
                 * @type Object
                 **/
                pcodeRulesOutcomes = {},

                /**
                 * Number of rule.js calls that are currently in flight
                 * @type Number
                **/
                rulesjsInFlight = 0,
                firstScriptTag = null,

                /**
                 * A map from pcode => Object
                **/
                defaultOptionsForPCodes = {},

                /**
                 * Current batch of pixel options that need to be enqueued.
                 *
                 * @type object {string: object}
                **/
                aggregatePixelBatch = {},

                /**
                 * This is set the first time a first-party cookie is set
                 */
                firstPartyCookieString = null,

                /**
                 * Utility functions
                 */
                slice = [].slice,

                isDst, tzOffset, escape, isDefined, random,
                getCookie, setFirstPartyCookie, removeCookie,
                init, doPush, generateFirstPartyParamStrings,
                generateFirstPartyCookieString, generateFirstPartyCookieValue,
                enqueuePixel, fire, fireAll, serializePixelParams, normalizeConfig,
                parseOpenGraphLabels, fetchRulesForPCode, loadScript,mergeOptions, doMerge,
                findInArray, findPCodes, consumeQEvents, toType, cloneQoptions, mergeRuleResult, sanitizeConfig,
                and, addMultipleRules, doAddRule, hasRules, processRule, evalRule, addPCodeToRulesBatch, setDefaults,
                addToBatch, flushBatch, insertRulesBundle, onRulesLoad, doOptionPush, initArrayLikeOptions, consumeQOptions
                , shouldSkipNormalizing;

            (function () {
                var elem;
                elem = doc.createElement('script');
                if ('async' in elem) {
                    SCRIPT_LOADER = SCRIPT_LOADER_MODERN;
                } else if (elem.readyState) {
                    SCRIPT_LOADER = SCRIPT_LOADER_LEGACY;
                } else {
                    SCRIPT_LOADER = SCRIPT_LOADER_UNSUPPORTED;
                }
                elem = null;
            })();

            /**
             * This is the heart and entry-point of quant.js
             * This replaces (or atleast provides alternative) to the now deprecated quantserve() function:
             *    • __qc.qhash(n) can now be achieved using __qc('hash', n)
             *    • Invoke __qc('init') instead of quantserve() to initialize quantjs
             *
             * @function
             *
             **/
            var api = function(cmd) {
                try {
                    return {
                        init: init,
                        hash: hashing.SHA256,
                        push: doPush,
                        rules: addMultipleRules,
                        require: require,
                        hasRules: hasRules,
                        defaults: setDefaults,
                        __qc: function(){ return true;}
                    }[cmd].apply(null, slice.call(arguments, 1));
                } catch(nfe) {
                    log.error(nfe);
                    return false;
                }
            };

            api.evts = 0;

            // QuantJS version
            api.v = 2.0;

            // List of ad domains
            api.SD = SD;

            /**
             * Array containing all the p-codes for which pixels have been fired
             * @legacy Gizmodo uses this to fire multiple pixels. Retain this for legacy
             *
             * @type Array<string>
            **/
            api.qpixelsent = [];

            /*
             * __qc('p-c0de', 'rule', reducerFn, actionFn, conditionsList); adds a rule to the system
             * where `conditionsList` is an Array: [conditionFn, ...args].
             *
             * System calls conditionFn with a callback and some transposed args.
             *
             * For example, in case of 'click', the element of the conditionsList would look like:
             * [eventFn, 'foo', 'click']
             * eventFn should be reference to a function which accepts a (callback, 'foo', 'click').
             * It is expected that eventFn should invoke callback() when the event (in this case, a mouse click)
             * takes place.
            */

            /**
             * Logical AND for condition list
             *
             * @function
             * @param conditions Array<object> List of conditions that need to be evaluated
             *
             * @return bool
            **/
            and = function(conditions) {
                var len = conditions ? conditions.length || 0 : 0,
                    i;

                for (i = 0; i < len; i++) {
                    if(!conditions[i]) {
                        return false;
                    }
                }

                return true;
            };

            addPCodeToRulesBatch = function(pcode) {

                pcode = pcode || win._qacct;

                if (!pcode) {
                    return;
                }

                if (!findInArray(pcodesFound, pcode)) {
                    pcodesFound.push(pcode);
                }

            };

            findInArray = function (arr, val) {
                var len = arr.length,
                    i;

                for (i = 0; i < len; i++) {
                    if (arr[i] === val) {
                        return true;
                    }
                }

                return false;
            };

            toType = function (obj) {
                return ({}).toString.call(obj).match(/\s([a-zA-Z]+)/)[1].toLowerCase();
            };

            cloneQoptions = function (obj) {
                var clone, type, k;

                type = toType(obj);

                if (type === 'array') {
                    return obj.slice(0);
                } else if (type === 'object') {
                    clone = {};

                    for (k in obj) {

                        if (obj.hasOwnProperty(k)) {
                            clone[k] = obj[k];
                        }
                    }

                    return clone;

                } else if (type === 'string') {
                    return '' + obj;
                }

                return obj;

            };

            doPush = function (opt, force, target) {
                enqueuePixel(opt, force, target);
            };

            /**
             * Check if we already have rules for a particular p-code
             * @param String pcode
             *
             * @return boolean true if rules exist.
             **/
            hasRules = function (pcode) {

                return findInArray(pCodesWithRules, pcode);
            };

            setDefaults = function (pcode, opts) {

                var curr;

                if (!pcode) return;

                curr = defaultOptionsForPCodes[pcode];
                if (curr) {
                    opts = mergeOptions(opts, curr);
                }

                if (opts.qacct) {
                    delete opts.qacct;
                }

                defaultOptionsForPCodes[pcode] = opts;
            };

            /**
             * Accepts an non-array-like object like _qoptions and aggregates them into the current batch
             *
             * @function
             * @param opt object
             *
            **/
            addToBatch = function(opt) {
                var event, media,
                    p, k, configOptions, pcode;

                if (!isDefined(opt)) {
                    return;
                }

                configOptions = opt;

                /* Since _qoptions is an object or array-like,
                 * the only way to distinguish them is by checking the type of the values
                 *
                 * NOTE: Don't use array for-loop.
                 * It would break for when its an object literal not array
                 */
                for (k in configOptions) {

                    if (typeof (configOptions[k]) == 'string') {
                        /* If the elements of qopts are strings, it probably means
                        * we have a reference to object literal.
                        */
                        event = opt.event || EVENT_TYPE_LOAD;
                        media = opt.media || MEDIA_TYPE_WEBPAGE;

                        /**
                         * We try our 'best effort batching' only for the p-codes found in the first event loop.
                         * We batch opts with event or media with special meaning.
                        **/
                        if (!(event === RULE_EVENT_TYPE || event === EVENT_TYPE_LOAD) ||
                            !(media === MEDIA_TYPE_WEBPAGE || media === MEDIA_TYPE_AD)) {
                            enqueuePixel(opt);
                        } else {
                            pcode = opt.qacct || win._qacct;
                            opt.qacct = pcode;

                            p = aggregatePixelBatch[pcode];
                            if (p) {
                                p = mergeOptions(p, opt);

                            } else {
                                p = opt;
                            }
                            aggregatePixelBatch[pcode] = p;
                        }

                        addPCodeToRulesBatch(opt.qacct);
                        break;

                    } else if (typeof (configOptions[k]) == 'object' && configOptions[k] != null) {
                        addToBatch(configOptions[k]);
                    }
                }
            };

            /**
             * Merges two quantjs options objects
             *
             * @param o1 object
             * @param o2 object
             *
             * @returns object Merged options
             **/
            mergeOptions = function(o1, o2) {
                var merged = {};

                merged.qacct = o1.qacct || o2.qacct;

                if (o1.event === EVENT_TYPE_LOAD || o2.event === EVENT_TYPE_LOAD) {
                    merged.event = EVENT_TYPE_LOAD;
                } else if (!o1.event || !o2.event) {
                    merged.event = null;
                } else {
                    merged.event = o1.event || o2.event;
                }


                merged.media = null;

                if (o1.media === MEDIA_TYPE_WEBPAGE || o2.media === MEDIA_TYPE_WEBPAGE) {
                    merged.media = MEDIA_TYPE_WEBPAGE;
                } else if (o1.media === MEDIA_TYPE_AD || o2.media === MEDIA_TYPE_AD) {
                    merged.media = MEDIA_TYPE_AD;
                } else {
                    merged.media = o1.media || o2.media;
                }

                doMerge(merged, o1, o2);
                doMerge(merged, o2, o1);

                if (!merged.event) {
                    delete merged.event;
                }

                if (!merged.media) {
                    delete merged.media;
                }

                return merged;
            };

            doMerge = function (merged, o1, o2) {
                var k, p1, p2, m, f1, f2;

                for (k in o1) {

                    if (o1.hasOwnProperty(k) && !merged.hasOwnProperty(k)) {
                        p1 = o1[k];
                        p2 = o2[k];
                        m = '';

                        f1 = !!p1 && typeof p1 == 'string';
                        f2 = !!p2 && typeof p2 == 'string';

                        if (f1) {
                            m = p1;
                        }

                        if (f1 && f2) {
                            m += ',';
                        }

                        if (f2) {
                            m += p2;
                        }

                        merged[k] = m;
                    }
                }
            };

            flushBatch = function() {
                var batch = [],
                    k, pixel;

                // If some rules are being fetched, we can wait.
                // flushBatch() will be invoked after fetch of those rules
                if (rulesjsInFlight > 0) {
                    return;
                }

                consumeQEvents();

                for (k in aggregatePixelBatch) {
                    if (aggregatePixelBatch.hasOwnProperty(k) && aggregatePixelBatch[k]) {
                        pixel = aggregatePixelBatch[k];
                        batch.push(pixel);
                        delete aggregatePixelBatch[k];
                    }
                }

                if (batch.length == 1) {	
                    enqueuePixel(batch[0]);	
                }	
                if (batch.length > 1) {
                    if (ALLOW_MULTI_PIXEL) {
                        enqueuePixel(batch);
                    } else {
                        for (k = 0; k < batch.length; k++) {
                            enqueuePixel(batch[k]);
                        }
                    }
                }
            };

            insertRulesBundle = function() {

                var pCodesToFetch = [],
                    i, p, pcodes;

                pcodes = pcodesFound.slice(0);

                for (i = 0; i < pcodes.length; i++) {
                    p = pcodes[i];
                    if (!hasRules(p)) {
                        pCodesToFetch.push(p);
                    }
                }


                if (pCodesToFetch.length === 0) {

                    // Incase of aquant, we will find that the rules already exist
                    // Hence no rule bundle will be inserted and flushBatch will never be fired
                    // We hence need to force a flushBatch() if it has never been flushed before
                    flushBatch();

                } else {
                    for (i = 0; i < pCodesToFetch.length; i++) {
                        p = pCodesToFetch[i];
                        pCodesWithRules.push(p);
                        fetchRulesForPCode(p);
                    }
                }

            };

            /**
             * @param {string} src Source URL (without scheme)
             * @param {function} onLoad Callback invoked when script loads
             * @param {function} onError Callback invoked when fetch request fails (optional)
             * @param {function} onUnsupported Callback invoked if dynamic script loading is unsupported (optional)
             */
            loadScript = function (src, onLoad, onError, onUnsupported) {
                var elem;

                src = win.location.protocol + '//' + src;

                firstScriptTag = doc.scripts && doc.scripts[0] || null;
                var scriptParent = firstScriptTag && firstScriptTag.parentNode || doc.head || doc;

                elem = doc.createElement('script');

                if (SCRIPT_LOADER === SCRIPT_LOADER_MODERN) {
                    elem.src = src;
                    elem.async = true;
                    elem.onload = onLoad;
                    if (onError) {
                        elem.onerror = function (event) {
                            // avoid possibility of infinite loop
                            elem.onerror = null;

                            onError(event);
                        };
                    }

                    scriptParent.insertBefore(elem, firstScriptTag);

                } else if (SCRIPT_LOADER === SCRIPT_LOADER_LEGACY) { // IE < 10

                    var done = false;


                    elem.onload = elem.onreadystatechange = function () {


                        if (!done && (elem.readyState == 'loaded' || elem.readyState == 'complete')) {
                            done = true;
                            elem.onreadystatechange = null;
                            onLoad();
                        }
                    };

                    elem.src = src;

                    scriptParent.insertBefore(elem, firstScriptTag);

                } else {
                    if (onUnsupported) {
                        onUnsupported();
                    }
                }
            };

            fetchRulesForPCode = function (pcode) {
                rulesjsInFlight++;


                loadScript(
                    'rules.quantcount.com/rules-' + pcode + '.js',
                    function () {

                        pcodeRulesOutcomes[pcode] = SCRIPT_LOADER === SCRIPT_LOADER_LEGACY ? RULE_FETCH_INDETERMINATE : RULE_FETCH_OK;
                        onRulesLoad();
                    },
                    function (evt) {

                        pcodeRulesOutcomes[pcode] = RULE_FETCH_FAILED;
                        onRulesLoad();
                    },
                    function () {

                        //Set param to pixel call if we can't retrieve pcode
                        pcodeRulesOutcomes[pcode] = RULE_FETCH_UNSUPPORTED;
                        onRulesLoad();
                    }
                );
            };

            onRulesLoad = function() {
                // Decrement if greater than zero
                rulesjsInFlight -= rulesjsInFlight > 0 ? 1 : 0;

                flushBatch();
            };

            /**
             * This function is called when __qc('rules', ...); is invoked.
             * @function
            **/
            addMultipleRules = function() {
                var doBatch = true,
                    ruleTriggered = false,
                    i, args, callback;

                if (arguments.length) {
                    /* Two scenarios are possible:
                     *    1. Some callbacks might happen in the current event loop (URL rules)
                     *    2. Others (click & other event based rules) might happen at a later time
                     *
                     * For all the callbacks that return in current event loop,
                     * doBatch would be `true`.
                     * Since we reset doBatch later, other later returning
                     * callbacks see it as false.
                    */
                    callback = function(opt) {
                        if (doBatch) {
                            addToBatch(opt);
                        } else {
                            enqueuePixel(opt, true);
                        }
                        ruleTriggered = true;
                    };
                    for (i = 0; i < arguments.length; i++) {
                        args = slice.call(arguments[i], 0);
                        args.splice(1, 0, callback);
                        doAddRule.apply(null, args);
                    }
                    doBatch = false;

                    if (isInitialized) {
                        flushBatch();
                    }
                }
                return ruleTriggered;
            };

            /**
             * @param pCode string
             * @param batchingCallback function
             * @param conditionFn function
             * @param actionFn function
             *
            **/
            doAddRule = function(pCode, batchingCallback) {

                var flags = [],
                    values = [],
                    pixelCallback = batchingCallback || enqueuePixel,
                    len, params, reducerFn, actionFn,
                    conditions, i, rule;

                params = slice.call(arguments, 2);

                if (params && params.length) {


                    reducerFn = params[0] || and;
                    actionFn = params[1];
                    conditions = params[2];

                    len = conditions.length;

                    // Initialize flags and values for each condition
                    for (i = 0; i < len; i++) {
                        flags.push(false);
                        values.push(null);
                    }

                    rule = {
                        p: pCode,
                        f: flags, // Indicates which conditions have yielded a definite result
                        r: reducerFn,
                        c: conditions,
                        a: actionFn,
                        v: values // values returned by each of the conditions
                    };

                    if (!hasRules(pCode)) {
                        pCodesWithRules.push(pCode);
                    }

                    rules.push(rule);
                    processRule(rule, pixelCallback);

                }  else {
                    pCodesWithRules.push(pCode);
                    pcodeRulesOutcomes[pCode] = RULE_FETCH_EMPTY;
                }
            };

            /**
             * Call the condition functions by passing in the system's callbacks
             *
             * @function
             * @param rule object
             * @param pixelCallback function|null. If null, uses enqueuePixel.
             *
            **/
            processRule = function(rule, pixelCallback) {
                var len = rule ? rule.c ? rule.c.length : 0 : 0, // Safe-guard against nulls
                    i;

                for (i = 0; i < len; i++) {

                    /*eslint-disable no-loop-func*/
                    (function(j) { // IIFE to safe-guard against side-effects of closure over `i`
                        var conditionFn, args;
                        try {
                            conditionFn = rule.c[j][0];
                            args = rule.c[j].slice(1);

                            args.splice(0, 0, function(val) {
                                rule.f[j] = true;
                                rule.v[j] = val;

                                evalRule(rule, pixelCallback);
                            });
                            conditionFn.apply(null, args);
                        } catch (nfe) {
                            rule.f[j] = true;
                            rule.v[j] = false;
                            evalRule(rule, pixelCallback);
                        }

                    })(i);
                    /*eslint-enable no-loop-func*/
                }
            };

            evalRule = function(rule, pixelCallback) {
                var actionFns = rule.a,
                    flags = rule.f,
                    values = rule.v,
                    reducerFn = rule.r || and,
                    result, action, actionArgs, actionResult, i, opt, k;

                result = and(flags);

                if (result) { // All callbacks have returned
                    result = result && reducerFn(values);  //...and all are truthy
                    // This also means we only call reducer when all rules have returned
                }

                if (result) {

                    for (i = 0; i < actionFns.length; i++) {

                        try {
                            action = actionFns[i][0];
                            actionArgs = actionFns[i].length > 1 ? actionFns[i].slice(1) : [];
                            actionArgs = actionArgs.concat(rule.v);
                            actionResult = action.apply(null, actionArgs);

                            opt = {
                                qacct: rule.p,
                                event: RULE_EVENT_TYPE
                            };

                            for (k in actionResult) {
                                if (actionResult.hasOwnProperty(k) && k !== 'qacct') {
                                    opt[k] = actionResult[k];
                                }
                            }
                            // Force enqueuePixel so that pixel fires even if a pixel has previously fired for same p-code
                            pixelCallback(opt);
                        } catch (nfe) {
                            continue;
                        }
                    }

                }
            };

            /**
             *  Determine if day light savings needs to be applied
             *
             *  @function
             *  @return bool true if timezone uses daylight savings
            **/
            isDst = function() {
                return tzOffset(0) !== tzOffset(6) ? 1 : 0;
            };

            /**
            * Given a month, find the time zone offset
            *
            * @param month, number
            * @return tzOffset
            **/
            tzOffset = function(month) {
                var d1 = new Date(2000, month, 1, 0, 0, 0, 0);
                var t = d1.toGMTString();
                var d3 = new Date(t.substring(0, t.lastIndexOf(' ') - 1));
                return d1 - d3;
            };

            /**
             * encodeURIComponent for browsers that don't support it
             *
             * @function
             * @param string text to be encoded
             *
             * @return string URL encoded string
             **/
            escape = function(s) {
                return s.replace(/\./g, '%2E').replace(/,/g, '%2C');
            };

            /**
             * Is a particular variable defined?
             *
             * @param o any
             * @return bool, true if variable is defined
             **/
            isDefined = function(o) {
                return (typeof (o) != 'undefined' && o != null);
            };

            /**
            * Generate a random number for cache busting
            *
            * @return number
            **/
            random = function() {
                return Math.round(Math.random() * 2147483647);
            };


            /**
            * Retrieve a cookie
            *
            * @param cookieName string
            *
            * @return string value of the cookie
            **/
            getCookie = function(cookieName) {


                var value = '',
                    c = doc.cookie,
                    i, offset, end;

                if (!c) {

                    return value;
                }

                i = c.indexOf(cookieName + '=');
                offset = i + cookieName.length + 1;

                if (i > -1) {
                    end = c.indexOf(';', offset);
                    if (end < 0) {
                        end = c.length;
                    }
                    value = c.substring(offset, end);
                }


                return value;
            };

            /**
             * @param {number} time The timestamp with which to generate the cookie value
             */
            generateFirstPartyCookieValue = function(time) {
                return 'P0-' + random() + '-' + time.getTime();
            };

            /**
             * @param {string} qcCookie __qca cookie value
             * @param {Date} expiry JavaScript date expiry
             * @param {string} domain Current domain
             */
            generateFirstPartyCookieString = function(qcCookie, expiry, domain) {
                return [
                    '__qca=',
                    qcCookie,
                    '; expires=',
                    expiry.toGMTString(),
                    '; path=/; domain=',
                    domain
                ].join('');
            };

            /**
             * Generate first party param strings.
             *
             * @return An array containing three strings.
             *
             * At index 0: First party pixel param string (fpan, fpa).
             * At index 1: A cookie string that can be used to set document.cookie
             *             or an empty string if a first-party cookie should not be set.
             * At index 2: The param string for the sharedID/pubcommonId cookie, if present (pbc).
             */
            generateFirstPartyParamStrings = function() {
                var strings = ['', '', ''],
                    domainHash, nowEpoch, qcCookie, i,
                    pubcidCookie, pubcidOptoutCookie, sharedIDCookie, prebidOptoutCookie, prebidOptout;

                if (isAd === 1) {

                    strings[0] = ';fpan=u;fpa=';
                    return strings;
                }

                domainHash = hashing.FNV(domain);
                for (i = 0; i < SD.length; i++) {
                    if (SD[i] === domainHash) {

                        strings[0] = ';fpan=u;fpa=';
                        return strings;
                    }
                }

                nowEpoch = new Date(); // Grab today's date
                qcCookie = getCookie('__qca'); // Retrieve our first party cookie


                if (qcCookie.length > 0 || firstPartyCookieString) {

                    if (qcCookie.length === 0) {
                        // use cached first-party cookie string if actual first-party cookie has not yet been set
                        qcCookie = firstPartyCookieString;
                        strings[1] = generateFirstPartyCookieString(firstPartyCookieString, new Date(nowEpoch.getTime() + COOKIE_EXP_TIME), domain);
                    }

                    strings[0] = ';fpan=0;fpa=' + qcCookie;

                } else {
                    firstPartyCookieString = generateFirstPartyCookieValue(nowEpoch);

                    strings[1] = generateFirstPartyCookieString(
                        firstPartyCookieString,
                        new Date(nowEpoch.getTime() + COOKIE_EXP_TIME),
                        domain
                    );

                    strings[0] = ';fpan=1;fpa=' + firstPartyCookieString;
                }

                // We attempt to load a pubcommonID from default/well known first party cookie value
                // See https://docs.prebid.org/dev-docs/modules/pubCommonId.html
                pubcidCookie = getCookie('_pubcid'); // Try to retrieve the pubcid cookie
                pubcidOptoutCookie = getCookie('_pubcid_optout'); // Retrieve pubcid optout info

                // If we didn't find a _pubcid cookie, then look for a sharedID one.
                // Since pubcommonID merged with sharedID we also check for the value in the newer default _sharedID cookie
                // See https://docs.prebid.org/identity/sharedid.html
                sharedIDCookie = pubcidCookie.length > 0 ? pubcidCookie : getCookie('_sharedID');

                // Retrieve optout info See:
                // See https://docs.prebid.org/identity/sharedid.html#opt-out
                // and https://docs.prebid.org/dev-docs/modules/userId.html#publisher-first-party-opt-out
                prebidOptoutCookie = getCookie('_pbjs_id_optout');

                // If the prebid opt out cookie exists with any value we should assume the user is opted out
                // alternatively, if the pubcommon optout exists with a value that is '1' consider them opted out
                prebidOptout = (prebidOptoutCookie.length > 0 || pubcidOptoutCookie === '1');

                if (!prebidOptout && sharedIDCookie.length > 0) {
                    // The user has not opted-out of using the sharedID, and a sharedID cookie is present
                    strings[2] = ';pbc=' + sharedIDCookie;
                } else {
                    strings[2] = ';pbc=';
                }

                return strings;
            };

            /**
             * Set a first party quantcast cookie (__qca)
             **/
            setFirstPartyCookie = function() {

                var qcaCookieString = generateFirstPartyParamStrings()[1];
                if (qcaCookieString) {

                    doc.cookie = qcaCookieString;
                }

            };

            /**
             * Try to purge the cookie by setting expiry to 1970
             *
             * @param n string name of cookie
             */
            removeCookie = function(n) {


                doc.cookie = n +
                    '=; expires=Thu, 01 Jan 1970 00:00:01 GMT; path=/; domain=' + domain;


            };

            sanitizeConfig = function (opt) {

                var k, i;

                if (opt && toType(opt) === 'object') {
                    for (i = 0; i < RESERVED_FIELDS.length; i++) {
                        k = RESERVED_FIELDS[i];

                        if (opt.hasOwnProperty(k) && opt[k]) {
                            delete opt[k];
                        }
                    }

                }
            };

            /**
             * Parses a QOptions object, merges with rule result, sanitizes and returns a new ClientOptions object
             *
             * @param suffix string Suffix to add to each key.
             *        Useful when there are multiple configs with possibly same keys
             * @param mayBeObject Object|null Object literal config
             * @param force boolean
             *
             * @returns string The normalized ClientOptions Object
             **/
            normalizeConfig = function (suffix, mayBeObject, force) {
                var pCode, ruleFetchOutcome, defaults;

                if (mayBeObject && typeof mayBeObject.qacct == 'string') {
                    pCode = mayBeObject.qacct;
                } else if (typeof win._qacct == 'string') {
                    pCode = win._qacct;
                }

                if (!pCode || pCode.length === 0) {
                    return null;
                }
                mayBeObject = mergeRuleResult(pCode, mayBeObject);
                delete aggregatePixelBatch[pCode];
                defaults = defaultOptionsForPCodes[pCode];

                ruleFetchOutcome = pcodeRulesOutcomes[pCode];
                if (!isDefined(ruleFetchOutcome)) {
                    ruleFetchOutcome = RULE_FETCH_UNKNOWN;
                }

                if (!shouldSkipNormalizing(mayBeObject, defaults, force, pCode)) {
                    return normalize(suffix, mayBeObject, defaults, pCode, ruleFetchOutcome);
                }
                return null;
            };

            /**
             * Accept either _qevents or _qoptions and create a param string for pixel call
             * If neither _qoptions nor _qevents have been passed, retrieve p-code using _qacct value
             *
             * @param suffix string Suffix to add to each key.
             *        Useful when there are multiple configs with possibly same keys
             *
             * @param mayBeObject Object|null Object literal config
             *
             *
             * @returns array[string] An array consisting of two elements:
             *          The serialized config (0), and the serialized email hash values (1)
             **/
            serializePixelParams = function(mayBeObject) {

                var serialized = [],
                    k, uhVals = [], serializedResult = [];

                for (k in mayBeObject) {

                    if (mayBeObject[k] && mayBeObject.hasOwnProperty(k)) {
                        if (k === 'uh' || k === 'uht') {
                            // A leading semicolon is necessary, since the leading semicolon
                            // is not added later.
                            uhVals.push(';' + k + '=' + mayBeObject[k]);
                        } else {
                            serialized.push(k + '=' + mayBeObject[k]);
                        }
                    }
                }

                serializedResult.push(serialized.join(';'));

                // uhVals already includes the semicolon between entries
                serializedResult.push(uhVals.join(''));

                return serializedResult;
            };

            /**
             * Retrieve and parse meta tags related to OpenGraph Protocol
             * {@linkplain http://ogp.me/}
             *
             * @return URI encoded OpenGraph values
             **/
            parseOpenGraphLabels = function() {
                var metaTags = doc.getElementsByTagName('meta'),
                    o = '',
                    i, p, c, l, m;

                for (i = 0; i < metaTags.length; i++) {
                    m = metaTags[i];

                    if (o.length >= 1000) {
                        return encodeURIComponent(o);
                    }

                    if (
                        isDefined(m) && isDefined(m.attributes) && isDefined(m.attributes.property) &&
                        isDefined(m.attributes.property.value) && isDefined(m.content)
                    ) {
                        p = m.attributes.property.value;
                        c = m.content;
                        if (p.length > 3 && p.substring(0, 3) === 'og:') {
                            if (o.length > 0) {
                                o += ',';
                            }
                            l = (c.length > 80) ? 80 : c.length;
                            o += escape(p.substring(3, p.length)) + '.' + escape(c.substring(0, l));
                        }
                    }
                }
                return encodeURIComponent(o);
            };

            /**
             * Enqueue a pixel call that will be fired later
             *
             * @function
             *
             * @param qoptions Object|Array qoptions is an array when it is _qevents
             *        and object literal when it is _qoptions
             **/
            enqueuePixel = function(qoptions, force, target) {
                var r = random(),
                    qMetaData = '',
                    url = '',
                    ourl = '',
                    referrer = '',
                    isIFrame = '1',
                    pixelUrlParams = [],
                    isCookiesEnabled, openGraphLabels, configOptions, serializedQuantJSOptions,
                    i, now, dst, fpaUrlParams, pbcUrlParam, qType, qo, firstPartyParams;

                // this is reset to 0 on each call to accommodate multiple tags on the same page
                isAd = 0;

                if (!isDefined(api.qpixelsent)) {
                    api.qpixelsent = [];
                }
                if (isDefined(qoptions)) {

                    /* Since _qoptions is an object or array and _qevents is Array-like,
                     * the only way to distinguish them is by checking the type of the values
                     */
                    qType = toType(qoptions);

                    if (qType === 'object') {
                        configOptions = normalizeConfig('', qoptions, force);
                    } else if (qType === 'array') {
                        for (i = 0; i < qoptions.length; i++) {
                            qo = normalizeConfig('.' + (i + 1), qoptions[i], force);
                            if (i === 0) {
                                configOptions = qo;
                            } else {
                                configOptions = mergeOptions(configOptions, qo);
                            }
                        }
                    }


                } else if (typeof _qacct === 'string') {
                    configOptions = normalizeConfig('', null, force);
                }

                // If we did not get back any thing, we cant fire pixel
                if (!configOptions) {
                    return;
                }

                isCookiesEnabled = (nav.cookieEnabled) ? '1' : '0';

                /*
                 * Legacy: We also accept some other metadata through _qmeta:
                 *    _qmeta = "cWM6aWQ9ZmNkMjA4NDk1ZDU2NTt6PTk0MTA3O2E9MzI7Zz1tO2M9S1I=";
                 */
                if (isDefined(win._qmeta)) {
                    qMetaData = ';m=' + encodeURIComponent(win._qmeta);
                    win._qmeta = null;
                }

                now = new Date();
                dst = isDst();

                firstPartyParams = generateFirstPartyParamStrings();
                fpaUrlParams = firstPartyParams[0];
                pbcUrlParam = firstPartyParams[2];

                if (win.location && win.location.href) {
                    url = encodeURIComponent(win.location.href);
                }
                if (doc && doc.referrer) {
                    referrer = encodeURIComponent(doc.referrer);
                }
                if (win.self === win.top) {
                    isIFrame = '0';
                }

                if (!configOptions.url) {
                    configOptions.url = url;
                } else {
                    ourl = url;
                }

                if (!configOptions.ref) {
                    configOptions.ref = referrer || '';
                }

                openGraphLabels = parseOpenGraphLabels();

                serializedQuantJSOptions = serializePixelParams(configOptions);

                pixelUrlParams.push(
                    '/pixel;r=' +
                    r + ';' +
                    serializedQuantJSOptions[0]
                );

                pixelUrlParams.push(serializedQuantJSOptions[1]);

                pixelUrlParams.push(
                    fpaUrlParams +
                    pbcUrlParam
                );

                pixelUrlParams.push(
                    ';ns=' +
                    isIFrame +
                    ';ce=' +
                    isCookiesEnabled +
                    ';qjs=1' +
                    ';qv=' + QUANT_JS_VERSION
                );

                pixelUrlParams.push(
                    (!configOptions.ref ? ';ref=' : '') +
                    ';d=' + domain +
                    ';dst=' + dst +
                    ';et=' + now.getTime() +
                    ';tzo=' + now.getTimezoneOffset() +
                    (ourl ? ';ourl=' + ourl : '') +
                    qMetaData +
                    ';ogl=' + openGraphLabels
                );

                // We are sending the parameters as 5 parts because we will insert the consent
                // between parts 3 and 4 (0-indexed) and only add parts 0 and 1 when we have
                // consent.
                pixelQueue.push({
                    pixel: pixelUrlParams,
                    target,
                });    
                fireAll();
            };

            /**
             * Fire the pixel.
             *
             * @param pixel Array An array of 5 strings representing pixel params
             *
             **/
            fire = function(opts) {
                // OK ok ok...consentManager wasn't originally meant to be used
                // this way, maybe the API needs further refinement. This takes
                // advantage of the fact that it returns a promise that returns
                // the value of the callback *if* consent was present.
                // Eventually, hopefully things will be different, but there
                // are still likely to be alternate actions so perhaps
                // consentManager really isn't much but a function of a map and
                // fold.
                var pixel = opts.pixel;
                consentManager.consent(function() { return true; }).then(function(allowed) {
                    if (!allowed) {
                        removeCookie('__qca');
                    }
                    return (allowed) ? DOMAIN_QSERVE : DOMAIN_QCOUNT;
                }).then(function (target) {
                    var parameters = consentManager.parameters;

                    // Use a closure to obfuscate reading the email hashes and cookie value with consent
                    var getSerializedValues = (function() {
                        return function() {
                            if (target === DOMAIN_QSERVE) {
                                return [pixel[1], pixel[2]].join("");
                            }
                            return ";uh=u;uht=u";
                        };
                    })();

                    var fqtarget = isDefined(opts.target) ? opts.target : "https://pixel." + target;

                    return loader.image([
                        fqtarget, pixel[0], getSerializedValues(), pixel[3],
                        ";cm=", parameters["cm"],
                        (parameters["gdpr"] === 1) ? ";gdpr=1;gdpr_consent=" + parameters["gdpr_consent"] : ";gdpr=0",
                        (parameters["us_privacy"]) ? ";us_privacy=" + parameters["us_privacy"] : "",
                        pixel[4]
                    ].join("")).then(function(img) {
                        if (img && typeof (img.width) == 'number' && img.width === 3) {
                            removeCookie('__qca');
                        } else if (target === DOMAIN_QSERVE) {
                            // In order to be fully COPPA compliant we are required to avoid setting a
                            // 1st party cookie until after receiving a 1x1 pixel response.

                            setFirstPartyCookie();
                        }
                    });
                });
            };


            /**
             * Fire all pixels currently in the queue
             *
             * @function
             **/
            fireAll = function() {
                while (pixelQueue.length) {
                    fire(pixelQueue.shift());
                }
            };

            doOptionPush = function () {
                var args = arguments,
                    o, i;

                findPCodes([].slice.call(args));

                for (i = 0; i < args.length; i++) {
                    o = args[i];

                    enqueuePixel(o);
                }

                if (pcodesFound.length) {
                    insertRulesBundle();
                } else {
                    flushBatch();
                }
            };

            findPCodes = function (opts) {
                var t = toType(opts),
                    i;

                if (t === 'array') {
                    for (i = 0; i < opts.length; i++) {
                        findPCodes(opts[i]);
                    }
                } else if (t === 'object') {
                    addPCodeToRulesBatch(opts.qacct || win._qacct);
                }
            };

            consumeQEvents = function () {

                var k;

                // Check if _qoptions is defined. If it is, we will find qacct there
                if (!isQacctConsumed && !win._qevents.length && !win.ezt.length && typeof _qacct !== 'undefined') {
                    /* If _qoptions did not exist, qacct could only exist in _qevents array or _qacct.
                     * However, if _qevents is empty, we will probably find _qacct.
                     * Passing `null` to enqueuePixel will cause it to pick up _qacct.
                     */

                    enqueuePixel({
                        qacct: win._qacct
                    });
                    isQacctConsumed = true;
                }

                //If evts is 0, it would indicate we haven't consumed _qevents yet
                if (!api.evts) {
                    for (k in win._qevents) {
                        if (win._qevents[k] !== win._qevents.push && win._qevents.hasOwnProperty(k)) {

                            enqueuePixel(win._qevents[k]);
                        }
                    }

                    for (k in win.ezt) {
                        if (win.ezt[k] !== win.ezt.push && win.ezt.hasOwnProperty(k)) {

                            enqueuePixel(win.ezt[k]);
                        }
                    }

                    win._qevents = {
                        push: doOptionPush
                    };

                    win.ezt.push = function () {
                        var a = arguments,
                            i;
                        // queue manager exists for legacy reasons (handling of multiple aquants?)
                        if (isDefined(win.queueManager)) {
                            for(i = 0; i < a.length; i++){
                                win.queueManager.push(a[i]);
                            }
                        } else {
                            doOptionPush.apply(this, arguments);
                        }
                    };
                    api.evts = 1;
                }
            };

            consumeQOptions = function (opt) {
                var clone;

                if (opt) {
                    clone = cloneQoptions(opt);

                    findPCodes(opt);
                    win._qevents.push(clone);
                    opt = null;
                }
            };


            initArrayLikeOptions = function (seq) {

                seq.push = function () {
                    findPCodes([].slice.call(arguments));
                    insertRulesBundle();
                    return [].push.apply(seq, arguments);
                };
            };

            /**
             * Check the media and event type and sent a boolean indicator to skip the normalizing step if
             * the event type is load and the media type is a webpage and the pixel for that pCode has
             * already been fired or if force is set to false.
             *
             * Also, sets ad if the media is of ad type.
             *
             * @param mayBeObject Object|null Object literal config
             *
             * @param defaults Object|null Object literal config
             * @param force boolean
             *
             * @returns boolean indicate that normalizing should be skipped
             **/
            shouldSkipNormalizing = function (mayBeObject, defaults, force, pCode) {

                defaults = defaults || {};

                var media = ((mayBeObject) ? mayBeObject["media"] : defaults["media"]) || MEDIA_TYPE_WEBPAGE;
                var event = ((mayBeObject) ? mayBeObject["event"] : defaults["event"]) || EVENT_TYPE_LOAD;

                if (media === MEDIA_TYPE_AD) {
                    isAd = 1;
                }

                if (media === MEDIA_TYPE_WEBPAGE && event === EVENT_TYPE_LOAD) {
                    for (var i = 0; i < api.qpixelsent.length; i++) {
                        if (api.qpixelsent[i] === pCode && !force) {
                            return true;
                        }
                    }

                    api.qpixelsent.push(pCode);
                }
                return false;
            };

            mergeRuleResult = function (pCode, mayBeObject) {
                var ruleResult = aggregatePixelBatch[pCode];

                if (!mayBeObject) {
                    mayBeObject =  ruleResult;
                } else if (ruleResult) {
                    mayBeObject =  mergeOptions(mayBeObject, ruleResult);
                }
                sanitizeConfig(mayBeObject);
                return mayBeObject;
            };

            /**
             * Initialize, determine p-code and enqueue pixels
             *
             * Historically, we have supported 3 ways of giving us the p-code (among other info):
             *    1. _qevents.push({qacct:"p-vn4EUWUq3rLP9"});
             *    2. _qoptions = {qacct: "p-12345"};
             *    3. _qacct = "p-1235";
             *
             * We need to continue supporting all these methods for backward compat
             *
             * @alias quantserve();
             * @alias __qc('init');
             **/
            init = function() {


                // If _qevents is not defined, init it
                if (!isDefined(win._qevents)) {
                    win._qevents = [];
                }

                // ezt is an artifact from EZTag (it's virtually synonymous with qevent
                // s, but has some ezt-specific logic below).
                if (!isDefined(win.ezt)) {
                    win.ezt = [];
                }

                // _qoptions, qcdata, smarttagdata is deprecated, hence push it all to _qevents and discard
                consumeQOptions(win._qoptions);
                consumeQOptions(win.qcdata);
                consumeQOptions(win.smarttagdata);

                if (!api.evts) {
                    initArrayLikeOptions(win._qevents);
                    initArrayLikeOptions(win.ezt);
                }

                findPCodes(win.ezt);
                findPCodes(win._qevents);
                findPCodes({qacct: win._qacct});

                win._qoptions = null;

                /*
                 * The very first time, try to fetch a list of rules.
                 * In this case, flushBatch() happens once the rules.js is fetched.
                 * However, there can also be a case where the publishers loads new options objects,
                 * and fires quantserve() again. In that case, we need to flushBatch immediately.
                */
                if (pcodesFound.length) {
                    insertRulesBundle();
                } else {
                    flushBatch();
                }

                isInitialized = true;
            };

            win.quantserve = win.quantserve || init;
            api.quantserve = init;

            return api;

        })(window, window.document, window.navigator);
    }

    window.quantserve();

    qtrack();

    return window.__qc;
});

        // INSERT SCRIPT HERE
    }
})(window);
