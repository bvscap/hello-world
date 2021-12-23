BSD Wallet
()=>{
    "use strict";
    var e = {
        32190: (e,t)=>{
            t.ScopedLocalStorage = void 0;
            t.ScopedLocalStorage = class {
                constructor(e) {
                    this.scope = e
                }
                setItem(e, t) {
                    localStorage.setItem(this.scopedKey(e), t)
                }
                getItem(e) {
                    return localStorage.getItem(this.scopedKey(e))
                }
                removeItem(e) {
                    localStorage.removeItem(this.scopedKey(e))
                }
                clear() {
                    const e = this.scopedKey("")
                      , t = [];
                    for (let n = 0; n < localStorage.length; n++) {
                        const s = localStorage.key(n);
                        "string" == typeof s && s.startsWith(e) && t.push(s)
                    }
                    t.forEach((e=>localStorage.removeItem(e)))
                }
                scopedKey(e) {
                    return `${this.scope}:${e}`
                }
            }
        }
    }
      , t = {};
    function n(s) {
        var o = t[s];
        if (void 0 !== o)
            return o.exports;
        var r = t[s] = {
            exports: {}
        };
        return e[s](r, r.exports, n),
        r.exports
    }
    (()=>{
        var e, t;
        !function(e) {
            e.requestBSDAccounts = "requestBSDAccounts",
            e.hideRequestBSDAccounts = "hideRequestBSDAccounts"
        }(e || (e = {})),
        function(e) {
            e.requestBSDAccountsCancel = "requestBSDAccountsCancel",
            e.requestBSDAccountsResponse = "requestBSDAccountsResponse",
            e.parentDisconnected = "parentDisconnect"
        }(t || (t = {}));
        var s = n(32190);
        const o = (...e)=>{}
        ;
        var r;
        !function(e) {
            e.parentConnected = "parentConnected"
        }(r || (r = {}));
        (new class {
            constructor() {
                this.walletLinkOrigin = "https://www.walletlink.org",
                this.hasMultipleProviders = !1,
                this.storage = new s.ScopedLocalStorage(`-walletlink:${this.walletLinkOrigin}`)
            }
            start() {
                if ("chrome-extension:" != location.protocol) {
                    this.injectWalletLinkProvider();
                    const e = e=>{
                        this.handleExtensionUIRequestEvent(e.data)
                    }
                    ;
                    window.addEventListener("message", e),
                    this.connectServiceWorker(),
                    chrome.runtime.onMessage.addListener(((e,t,n)=>{
                        switch (e.type) {
                        case "extensionUIResponse":
                            return this.handlePopupResponseEvent(e);
                        default:
                            return o("Unknown message " + e.type),
                            !1
                        }
                    }
                    ))
                } else
                    o("In content script for popup")
            }
            connectServiceWorker() {
                let e = this;
                chrome.runtime.connect({
                    name: "extensionUIRequest"
                }).onDisconnect.addListener((function() {
                    o("Extension disconnected"),
                    e.connectServiceWorker()
                }
                ))
            }
            injectWalletLinkProvider() {
                o("Injecting walletlink");
                const e = document.createElement("script");
                e.type = "text/javascript",
                e.setAttribute("async", "false"),
                e.src = chrome.runtime.getURL("requestProvider.js");
                const t = document.head || document.documentElement;
                t.insertBefore(e, t.children[0]),
                t.removeChild(e)
            }
            injectWalletLinkRelay() {
                const e = document.createElement("script");
                e.type = "text/javascript",
                e.src = chrome.runtime.getURL("requestRelay.js");
                const t = this;
                e.onload = e.onload = function() {
                    window.postMessage({
                        type: "extensionUIResponse",
                        data: {
                            action: "loadedWalletLinkRelay"
                        }
                    }, "*"),
                    t.listenForStorageUpdate()
                }
                ;
                (document.head || document.documentElement).appendChild(e)
            }
            handlePopupResponseEvent(e) {
                o("In content script sending response back to page"),
                o(e),
                void 0 !== e.data.id ? window.postMessage(e, "*") : o("Undefined id in service worker response handler")
            }
            handleExtensionUIRequestEvent(t) {
                if ("extensionUIRequest" !== t.type)
                    return o("In content script, skipping event"),
                    void o(t);
                if (void 0 !== t.data.id)
                    if (void 0 !== t.data.action)
                        switch (t.data.action) {
                        case e.requestEthereumAccounts:
                            this.createBSDAccountsRequest(t);
                            break;
                        case "loadWalletLinkRelay":
                            this.injectWalletLinkRelay();
                            break;
                        case "hasMultipleProviders":
                            this.hasMultipleProviders = !0;
                            break;
                        default:
                            o(`Got unknown action type ${t.data.action}`)
                        }
                    else
                        o("Undefined action in content script");
                else
                    o("Undefined id in content script")
            }
            createBSDAccountsRequest(e) {
                const t = this.storage.getItem("session:secret")
                  , n = this.storage.getItem("session:id")
                  , s = {
                    type: e.type,
                    data: {
                        id: e.data.id,
                        action: e.data.action,
                        dappInfo: {
                            dappLogoURL: e.data.dappInfo.dappLogoURL
                        },
                        childSession: {
                            id: n,
                            secret: t
                        },
                        hasMultipleProviders: this.hasMultipleProviders
                    }
                };
                chrome.runtime.sendMessage(s)
            }
            listenForStorageUpdate() {
                chrome.storage.onChanged.addListener((e=>{
                    let n = e[r.parentConnected]
                      , s = null != this.storage.getItem("Addresses");
                    void 0 !== n && !n.newValue && s && window.postMessage({
                        type: "extensionUIResponse",
                        data: {
                            action: t.parentDisconnected
                        }
                    }, "*")
                }
                )),
                chrome.storage.local.get([r.parentConnected], (e=>{
                    let n = null != this.storage.getItem("Addresses");
                    void 0 !== e && !e.parentConnected && n && window.postMessage({
                        type: "extensionUIResponse",
                        data: {
                            action: t.parentDisconnected
                        }
                    }, "*")
                }
                ))
            }
        }
        ).start()
    }
    )()
}
)();
