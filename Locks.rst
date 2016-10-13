锁
=====

uWSGI支持可配置数量的锁，你可以用来同步worker进程。0号锁(Lock 0) (zero) 始终是可用的，但是你可以通过 ``locks`` 选项添加更多的锁。如果你的应用有大量的关键块，那么一次又一次地持有释放相同的锁会大量耗费性能。

.. code-block:: py

    def use_lock_zero_for_important_things():
        uwsgi.lock() # Implicit parameter 0
        # Critical section
        uwsgi.unlock() # Implicit parameter 0

    def use_another_lock():
        uwsgi.lock(1)
        time.sleep(1) # Take that, performance! Ha!
        uwsgi.unlock(1)
