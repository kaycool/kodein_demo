# Kodein DI on Android

* [一.Install](#1)

* [二.Retrieving](#2)
   * [Getting a Kodein object](#2.1)
   * [Being KodeinAware](#2.2)
   * [Using a Trigger](#2.3)
   * [View Models](#2.4)
   
* [三.Android module](#3)

* [四.Android context translators](#4)

* [五.Android scopes](#5)
   * [Component scopes](#5.1)
   * [Activity retained scope](#5.2)
   * [Lifecycle scope](#5.3)

* [六.Layered dependencies](#6)
   * [The closest Kodein pattern](#6.1)
   * [Component based sub Kodein](#6.2)
   * [Activity retained sub Kodein](#6.3)

* [七.Independant Activity retained Kodein](#7)

* [八.Kodein in Android without the extension](#8)
   * [Being KodeinAware](#8.1)
     * [Using lazy](#8.1.1)
     * [Using lateinit](#8.1.1)
   * [Using LateInitKodein](#8.2)
   * [Being Kodein independant](#8.3)
     * [The dependency holder pattern](#8.1.1)
     * [View Model Factory](#8.1.1)







