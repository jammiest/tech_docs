# 首页

```sql
SELECT SQL_CALC_FOUND_ROWS
	sch.id,
	t.id ticket_id,
	sch.city_id,
	sch.venue_id,
	sch.pic,
	sch.show_id,
	sch.schedular_name,
	MAX( sch.show_time ) AS max_show_time,
	MIN( sch.show_time ) AS min_show_time,
	sch.method,
	sch.sell_status,
	MIN( st.price ) AS min_price,
	MAX( st.price ) AS max_price,
	sch.pre_sell_start_time,
	sch.sell_start_time,
	sch.pre_sell_count_time,
	sch.sell_count_time,
	s.cate_parent_id,
	v.NAME AS v_name,
	c.NAME AS c_name,
	sch.city_id,
	sch.id,
	sch.show_time 
FROM
	juo_show_schedular sch
	INNER JOIN juo_show AS s ON sch.show_id = s.id
	INNER JOIN juo_show_ticket t ON t.schedular_id = sch.id
	INNER JOIN juo_venue AS v ON v.id = sch.venue_id
	INNER JOIN juo_city AS c ON c.id = sch.city_id
	LEFT JOIN juo_hots_recommend_config AS hc ON hc.show_id = sch.show_id 
	AND hc.city_id = sch.city_id 
	AND hc.venue_id = sch.venue_id
	LEFT JOIN juo_show_ticket AS st ON st.schedular_id = sch.id 
	AND st.state = 1 
	AND st.del_state = 1 
WHERE
	sch.state = 1 
	AND sch.del_state = 1 
	AND sch.is_abolish = 0 
	AND sch.id > 77983 
GROUP BY
	t.id,
	sch.venue_id 
ORDER BY
	sch.id ASC,
	hc.is_top DESC,
	hc.top_time DESC,
	hc.total_weight DESC,
	hc.first_show_time ASC,
	( sch.show_time >= UNIX_TIMESTAMP( ) ) DESC,
	( CASE WHEN sch.show_time >= UNIX_TIMESTAMP( ) THEN sch.show_time END ) ASC,
	( CASE WHEN sch.show_time < UNIX_TIMESTAMP( ) THEN sch.show_time END ) DESC 
	LIMIT 220;
```