# 數據庫MySQL

# **MySQL—Python篇**
## **Python連接MySQL資料庫**

第一次建立的資料庫host = “localhost”和user = “root” 這兩項會是一樣的，其中的database = “maxdb”這欄是選擇要使用哪一個database，如果想選擇database再填寫即可，而password = “password”這塊要填寫當初安裝MySQL資料庫時自己所設定的密碼。

    # Connect MySQL
    import mysql.connector
    maxdb = mysql.connector.connect(
      host = "127.0.0.1",
      user = "root",
      password = "password",
      database = "maxdb",
      )
    cursor=maxdb.cursor()
## **Python新增MySQL資料**

首先建立database可以參考#Create db的段落指令，
建立好後再來是新增裡面的table可以參考#Create table的段落指令，
是輸入欄位名稱和數據資料可以參考#Insert Multiple Records的段落指令，如果database和table有重複建立的話是無法運行的唷。

    # Create db
    cursor.execute("CREATE DATABASE maxdb")
    
    # Create table
    cursor.execute("CREATE TABLE users (name VARCHAR(255), age INTEGER(99), user_id INTEGER AUTO_INCREMENT PRIMARY Key)")
    
    # Insert Multiple Records
    sqlStuff = "INSERT INTO users (name, age) VALUES (%s,%s)"
    records = [("Steve", 24),
               ("Max", 25),
               ("Chang" ,26),]
    cursor.executemany(sqlStuff, records)
    maxdb.commit()
## **Python讀取MySQL資料**
****
**利用Python讀取MySQL的資料，要留意的是其中Where name Like ‘M%’，M%這部分是讀取name欄位內所有M開頭的名稱，如果要指定特定名稱的話，可以寫成Where name = ‘Max’即可。**

     # Read
    cursor.execute("SELECT * FROM users Where name Like 'M%'")
    result = cursor.fetchall()
    for row in result:
        print(row)
## **Python更新MySQL資料**
****
**利用Python更新MySQL的資料，其中WHERE user_id = 3，這部分是使用不重複的欄位(像是id)更新，如果用欄位name來更新資料的話，名稱可能重複所以要留意可能會有多欄位被修改。**

    # Update
    update_users = "UPDATE users SET age = 23 WHERE user_id = 3"
    cursor.execute(update_users)
    maxdb.commit()
## **Python刪除MySQL資料**

利用Python清理MySQL內table的資料，DROP TABLE的功能使用上要特別留心，是沒辦法復原資料的唷！

    # Delete
    delete_users = "DELETE FROM users WHERE user_id = 4"
    cursor.execute(delete_users)
    maxdb.commit()
    
    # Delete Drpo Table
    delete_table = "DROP TABLE IF EXISTS users"
    cursor.execute(delete_table)

