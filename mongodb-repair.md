#mongodb意外关闭后的修复方法

###出现问题

疯狂向Mongodb写入数据,服务器CPU 99%持续一分钟!无奈强杀mongodb进程(kill -9 pid:切记不可-9关闭进程，之后的mongodb启动后连接不了数据，就是因为强杀进程所致)！
**正确的关闭Mongodb** : db.shutdownServer()和kill -2 pid
    
###问题处理

    
 1. 强杀后

    /etc/init.d/mongodb start

    启动完了
        
        mongo
        use database
        show collections
    
    collections 为空，心中一凉, 妈的数据没了(之前看过黑mongodb文章，会有数据莫名丢失的情况)

 2. 立马看了下dbpath文件夹下，文件实体是在的, 这样的话可能数据损坏!看一下log
    log 一直报 找不到用户名 auth failure, 用户的数据库没有，所以会验证失败!现在需要做的就是**repair**

 3. repair 

	[官方文档](http://docs.mongodb.org/manual/tutorial/recover-data-following-unexpected-shutdown/)
        
        官方文档里说: 如果你的mongod实例没有 加--journal（默认的journal是开启的） 而且没有运行**relication** 。你应该在开始mongodb之前运行repair操作!如果你使用replication，需要从备份中回复
		
        如果mongod.lock(非0字节)在/data/db 文件夹下，你需要删除mongod.lock  
		
        移除mongod.lock后  有两种方法去repair 
								                      
        1. 修复时使用**--repairpath**参数去修复(这种修复方式是相对安全的)
			
            首先在你的dbpath(/srv/db/mongodb)文件夹下新建一个backup文件夹(我这里就是/srv/db/mongodb/backup)这里如果不在这个dbpath文件夹下建立文件夹，修复时会提示错误
            这个backup文件夹是临时文件夹 mongod 修复时会在这个文件夹下生成数据库的临时文件，修复完成后会自动清楚(所以修复完此文件为空不要奇怪)

            mongod --dbpath /srv/db/mongodb --repair --repairpath /srv/db/mongodb/backup
            mongod --dbpath /srv/db/mongodb 就可以正常启动了(这里我运行/etc/init.d/mongodb start 就不能启动起来看log: permission error,意思时我的自启动脚本是mongodb用户启动，但是我现在已root用户去修复数据库，数据可的权限发生改变，所以此时需要 chown mongodb:mongodb -R /srv/db/mongodb )
	       
        2. 使用--repair (这种修复就是直接将恢复后的数据覆盖原来的数据,此时有个问题就是如果repair过程出现异常，可能原来的数据就会破坏)
            mongod --dbpath /srv/db/mongodb --repair
            mongod --dbpath /srv/db/mongodb 

