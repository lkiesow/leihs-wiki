## leihs -- the equipment booking and inventory management system

leihs is a system with which universities or other organizations can **manage and book their equipment**. People can place reservations/bookings on items and then pick them up at e.g. a reservation desk. The system keeps track of who has which items and gives equipment managers time to worry about other things.


## Documentation

* **Installation and administration:** To install leihs, you need a GNU/Linux server capable of running Ruby and Ruby on Rails. The installation itself is covered in the [[leihs-Admin-Guide]].

* **Production servers:** The guide above shows only a simple setup. If you want a production-class server using automated deployment, Phusion Passenger, Apache, Node.js etc., you can find a long and detailed example of how to set up something like that in the [[leihs-Production-Server-Guide]].

* **Development:** You can extend and adapt leihs to your needs. Some specific topics (such as how to help us translate leihs) are covered in the [[leihs-Developer-Guide]]. Also, see **[[Contributing]]** for information about how to get your contributions back into the project.

* **User guide:** There is no user guide for leihs yet. Do you want to write one? Feel free to start a new page on this wiki.

## Live demo

You can try out a live installation of leihs at http://demo.leihs.zhdk.ch. You can also try the new leihs 3 alpha at http://new.demo.leihs.zhdk.ch, but please note that only manager users are valid there. Normal users don't work on leihs 3.0 at the moment. If the demo happens to be down, please contact Ramon at ramon.cahenzli@zhdk.ch.

Demo data is deleted and reset every 24 hours.

### Manager
Can manage inventory, hand over items, create contracts, take back equipment and much more. He is mostly active in the backend, that's why he doesn't need frontend access.

Username: manager_user, password: pass
### Normal user
Can book equipment through the frontend and has no backend access.

Username: normal_user, password: pass

### Admin user
Can manage the system, create system-wide categories, create and remove new inventory pools.

Username: super_user_1, password: pass


## Most important links

 * [Downloadable packages](https://github.com/zhdk/leihs/releases) of major releases (2.0, 2.1 etc.). It's often better to get the latest versions of leihs through git, though.
 * [Leihs blog](http://blog.zhdk.ch/leihs), where end users can get news about what's going on in the project.
 * [Issues/discussion](http://github.com/zhdk/leihs/issues), where you can discuss new feature ideas and report bugs.
 * [Pivotal Tracker page](http://www.pivotaltracker.com/projects/130496) Where we plan our work and decide when things get done.
 * Chat with us on IRC, channel #leihs on [Freenode](http://freenode.net/). If you don't have an IRC client, you can use [webchat](https://webchat.freenode.net/?channels=#leihs).

## References (Who uses leihs?)

leihs is in use at several universities and organizations:

 * [Zürich University of the Arts (ZHdK)](http://www.zhdk.ch), Zürich, Switzerland
 * [Bern University of the Arts (BUA/HKB)](http://hkb.bfh.ch), Bern, Switzerland
 * [Berliner Technische Kunsthochschule (btk)](http://www.btk-fh.de/), Berlin, Germany
 * [University of Applied Sciences HTW Chur](http://www.fh-htwchur.ch), Switzerland
 * [Lucerne University of Applied Sciences and Arts](http://www.hslu.ch), Switzerland
 * University of Brighton, United Kingdom
 * University of Worcester, United Kingdom. 3 instances of leihs
 * Your university? Send a note to [Ramón Cahenzli](mailto:ramon.cahenzli@zhdk.ch) if you would like to be listed here

## Versioning

We subscribe to the [Semantic Versioning](http://semver.org/) approach.

## License

leihs is [Free Software](http://www.gnu.org/philosophy/free-sw.html), licensed under the [GNU General Public License (GPL) version 3.0](http://www.gnu.org/licenses/gpl-3.0.txt).
