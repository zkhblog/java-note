更新操作时：
①post请求，在带上_update时，会对比更新前数据和更新后数据，与原来一样就什么都不做，version不增加，seq_no也不增加  
不带_update时，会直接更新成功，version会增加  
②