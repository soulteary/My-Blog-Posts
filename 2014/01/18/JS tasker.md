# JS tasker

写了一个简单的js任务执行工具(publish/subscribe)

- GitHub地址：https://github.com/soulteary/js-tasker

```js
(function (win, $, undefined) {
    "use strict";

    var DEBUG = true;
    var VER = '20140118';
    if (!('console' in window)) {
        win.console = function () {
            return true;
        };
    }
    //用处理计划任务的方式去处理页面中的事物，这些解耦或许会好点？
    //有的时候我们会添加很多任务，但是不一定要立即执行，有的可能会先添加，然后等待某一个事件添加完成后才全部执行。
    //但是如果有的数据异常，之前的事情不想做了，那么也可以删除队列。
    win.task = {
        depend: {},
        __check__: function (cmd) {
            var protectList = ['__check__', 'exec', 'kill', 'add'];
            if ($.inArray(cmd, protectList) !== -1) {
                throw Error('THIS METHOD DO NOT ALLOW REWRITE.');
                return false;
            } else {
                return true;
            }
        },
        exec: function (cmd, title) {
            if (!this.__check__(cmd)) {
                return false;
            }
            if (cmd in this) {
                for (var i = 0, j = this[cmd].length; i < j; i++) {
                    //有依赖先执行依赖策略
                    if (this.depend[cmd] && this.depend[cmd]['mask']) {
                        $(this.depend[cmd]).each(function (k, v) {
                            if ($.inArray(i, v['mask']) !== -1) {
                                if (v['host'] !== undefined) {
                                    if (DEBUG) {
                                        console.log('#任务正在执行：', title);
                                    }
                                    if (task[cmd][v['host']] !== task[cmd][v['mask']]) {
                                        task[cmd][v['host']] && task[cmd][v['host']].call(),
                                            task[cmd][v['mask']] && task[cmd][v['mask']].call(),
                                            task.kill(cmd, v['mask']),
                                            task.kill(cmd, v['host']);
                                    } else {
                                        task[cmd][v['host']] && task[cmd][v['host']].call(), task.kill(cmd, v['host']);
                                    }
                                }
                            }
                        });
                    } else if (this.depend[cmd] && !this.depend[cmd]['mask']) {
                        this.depend[cmd]['host']
                        if (this.depend[cmd]['host'] !== undefined) {
                            if (DEBUG) {
                                console.log('#任务正在执行：', title);
                            }
                            task[cmd][this.depend[cmd]['host']] && task[cmd][this.depend[cmd]['host']].call(), task.kill(cmd, this.depend[cmd]['host']);
                        }
                    } else if (!this.depend[cmd]) {
                        if (DEBUG) {
                            console.log('#任务正在执行[无依赖]：', title);
                        }
                        this[cmd][i] && this[cmd][i].call(), this.kill(cmd, i--), j--;
                    } else {
                        throw Error('WAITING FOR DEPEND FUNC REG.');
                    }
                }

            }
        },
        add: function (options) {
            var cmd = options.cmd;
            var func = options.func || function () {
            };//默认空事件
            var trigger = options.trigger || false;   //默认事件不触发
            var option = options.option;
            var title = options.title || '';

            if (!cmd || !this.__check__(cmd)) {
                return false;
            }
            task[cmd] = task[cmd] || [];
            task[cmd].push(func);
            if (DEBUG) {
                console.log('#任务已添加：', title);
            }
            var exec = function (cmd) {
                if (cmd in task) {
                    task.exec(cmd, title);
                }
            };
            if (option) {
                var format = option.match(/(\w+):(\w+)/ig);
                if (format) {
                    var data = option.split(':');
                    var action = data[0];
                    var assign = data[1];
                    switch (action) {
                        case 'REG':
                            //如果没有注册过就注册，注册过就更新依赖的内容
                            if (DEBUG) {
                                console.log('#更新依赖：', title);
                            }
                            this.depend[assign] = this.depend[assign] || {};
                            if (this.depend[assign]['host'] && task[cmd][this.depend[assign]['host']]) {
                                this.kill(task[cmd], this.depend[assign]['host'])
                            }
                            this.depend[assign]['host'] = $.inArray(func, task[cmd]);
                            if (trigger === true) {
                                exec(cmd);
                            }
                            break;
                        case 'DEP':
                            //添加任务的时候，如果存在依赖的任务就执行掉。
                            //否则注册这个事件，但是主题执行的留空
                            if (DEBUG) {
                                console.log('#调用依赖：', title);
                            }
                            if (this.depend[assign]) {
                                task[cmd][this.depend[assign]].call(null);
                                delete this.depend[assign];
                                //这里或许需要把关联数组内的函数也全部执行一遍
                                if (trigger === true) {
                                    exec(cmd);
                                }
                            } else {
                                this.depend[assign] = {};
                                this.depend[assign]['mask'] = this.depend[assign]['mask'] || [];
                                this.depend[assign]['mask'].push($.inArray(func, task[cmd]));
                            }
                            break;
                    }
                }
            } else {
                if (trigger === true) {
                    exec(cmd);
                }
            }
        },
        kill: function (cmd, i) {
            if (!this.__check__(cmd)) {
                return false;
            }
            task[cmd].splice($.inArray(task[cmd][i], task[cmd]), 1);
            if (!task[cmd].length) {
                delete task[cmd];
            }
            if (DEBUG) {
                console.log('#删除了一个任务。');
            }
        },
        version:VER
    };
})(window, jQuery);

//SETTIMEOUT 模拟和服务器交互的接口的延迟，异步执行，等待执行。
//1.首先添加一个睡懒觉的任务，这个任务不会主动触发 trigger: false
//并且没有任何依赖 option: false
setTimeout(function () {
    var taskContent = {
        title: '1.想要睡懒觉',
        cmd: 'notice',
        trigger: false,
        option: false,
        func: function () {
            console.log('  >>>明天睡懒觉，这个事件被其他的事件触发了。');
        }
    }
    task.add(taskContent);
}, 100);
//2.然后添加一个计划起床的任务，这个任务会主动触发 trigger: true,
//并且在触发的时候将之前同组没有触发的任务也触发掉
setTimeout(function () {
    var taskContent = {
        title: '2.计划起床',
        cmd: 'notice',
        trigger: true,
        option: false,
        func: function () {
            console.log('  >>>计划有变，早点起床，这个任务会触发所有睡觉相关任务。');
        }
    }
    task.add(taskContent);
}, 300);
//3.接着添加一个出门前穿衣服的计划，这个计划有依赖，依赖为getup。
//即使trigger: true，如果没有依赖存在，他是不会执行的
//注意cmd和option中的事件需要一致，比如都是getup
setTimeout(function () {
    var taskContent = {
        title: '3.声明出门前穿衣服',
        cmd: 'getup',
        trigger: true,
        option: 'DEP:getup',
        func: function () {
            console.log('  >>>出门，但是要先穿上衣服。');
        }
    }
    task.add(taskContent);
}, 500);
//4.最后添加一个穿衣服的任务，这个任务注册事件为getup，会执行所有的依赖这个家伙的任务
setTimeout(function () {
    var taskContent = {
        cmd: 'getup',
        func: function () {
            console.log('  >>>起床，穿上衣服。');
        },
        trigger: true,
        option: 'REG:getup',
        title: '4.最后注册起床穿衣服'
    }
    task.add(taskContent);
}, 1500);
//5.随便注册一个函数并执行，没有依赖也无所谓
setTimeout(function () {
    var taskContent = {
        cmd: 'nerd',
        trigger: true,
        func: function (e) {
            console.log('  >>>发呆。')
        },
        option: 'REG:nerd',
        title: '5.发呆....'
    }
    task.add(taskContent);
}, 1500);
```

