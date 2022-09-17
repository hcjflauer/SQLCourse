# SQLCourse

Hi! Submitting my final in Mode report format did not work because I can't set the workspace to public.  The settings page where that change would 
be made is currently down.  Believe me when I tell you the Mode report exists and I've spent a frustrating amount of time trying to make it viewable.  
Below you will see 1) an approximation of the report (visualize the graphs in your imagination) and 2) the source code if you want to cut and paste
them into Mode to test them yourself and check the data snips that are in the report.

# Effects of Test 2 on Orders and Views - Report

1. Does the final_assignments_qa table have everything needed to compute a 30-day view-binary?

No, we need the assignment time and will need to connect to the orders table to compute the metrics measuring the impact of the tests

2. Write a query and table creation statement to make final_assignments_qa look like the final_assignments table.
Add in missing info with a placeholder of the appropriate data type.

  item_id   test_assignment   test_id   test_start_date                        
1	2512        	1	              a	      2022-09-16 00:00:00
2	482	          0	              a	      2022-09-16 00:00:00
3	2446        	0	              a	      2022-09-16 00:00:00
4	1312        	0	              a	      2022-09-16 00:00:00
...


3. Calculate the order binary for the 30-day window after the test assignment for test 2

  test_assignment   items   items_with_orders   items_with_recent_orders    percentage_recent_orders_per_assigned_item
1	    0	            1130	    1130	              341	                          30
2	    1	            1068	    1068	              319	                          29

I made a bar graph of the percentage of recent orders for control and treatment in test 2

4. Calculate the view binary and average views for the 30-day window after the test assignment for test 2

  test_assignment   items   items_with_views    items_with_recent_views   avg_recent_views    percentage_recent_views_per_assigned_item
1	    0	            1130	    1130	              918	                      97.5637168142	        81
2	    1	            1068	    1068	              890	                      97.4737827715	        83

I made a bar graph of the percentage of recent views for control and treatment in test 2


5. Compute the lifts and the p-values for the 30-day order and view binary metrics using a 95% confidence interval

order binary 
 lift: -1%
 p-value: 0.88
Test 2 did not have a significant impact on orders.

view binary 
 lift: 2.6%
 p-value: 0.2
Test 2 did not have a significant impact on views.


# Source Code

2. Reformat the data
SELECT 
    item_id
    ,test_a                               AS test_assignment
    ,'a'                                  AS test_id
    ,CAST('09-16-2022' AS timestamp)      AS test_start_date
  FROM 
    dsv1069.final_assignments_qa
UNION ALL
SELECT 
    item_id
    ,test_b                               AS test_assignment
    ,'b'                                  AS test_id
    ,CAST('09-16-2022' AS timestamp)      AS test_start_date
  FROM 
    dsv1069.final_assignments_qa
UNION ALL
SELECT 
    item_id
    ,test_c                               AS test_assignment
    ,'c'                                  AS test_id
    ,CAST('09-16-2022' AS timestamp)      AS test_start_date
  FROM 
    dsv1069.final_assignments_qa
UNION ALL
SELECT 
    item_id
    ,test_d                               AS test_assignment
    ,'d'                                  AS test_id
    ,CAST('09-16-2022' AS timestamp)      AS test_start_date
  FROM 
    dsv1069.final_assignments_qa
UNION ALL
SELECT 
    item_id
    ,test_e                               AS test_assignment
    ,'e'                                  AS test_id
    ,CAST('09-16-2022' AS timestamp)      AS test_start_date
  FROM 
    dsv1069.final_assignments_qa
UNION ALL
SELECT 
    item_id
    ,test_f                               AS test_assignment
    ,'f'                                  AS test_id
    ,CAST('09-16-2022' AS timestamp)      AS test_start_date
  FROM 
    dsv1069.final_assignments_qa
    
3. Compute Order Binary
SELECT 
    test_assignment
    ,count(item_id)                                             AS items
    ,sum(order_binary)                                          AS items_with_orders
    ,sum(order_binary_30d)                                      AS items_with_recent_orders
    ,sum(order_binary_30d)*100/count(item_id)                   AS percentage_recent_orders_per_assigned_item
  FROM
    (SELECT 
        assignments.item_id 
        ,assignments.test_number
        ,assignments.test_assignment 
        ,MAX( CASE 
              WHEN orders.created_at > assignments.test_start_date 
              THEN 1 
              ELSE 0 
              END)                                              AS order_binary
        ,MAX( CASE 
              WHEN (orders.created_at > assignments.test_start_date AND date_part('day', orders.created_at - assignments.test_start_date) <= 30)
              THEN 1
              ELSE 0
              END)                                              AS order_binary_30d
      FROM 
        (SELECT 
            *
          FROM
            dsv1069.final_assignments
          WHERE
            test_number = 'item_test_2'
        )                                                       AS assignments
      LEFT JOIN 
        dsv1069.orders
        ON assignments.item_id = orders.item_id 
      GROUP BY
        assignments.item_id 
        ,assignments.test_number
        ,assignments.test_assignment 
    )                                                           AS item_level
  GROUP BY
    test_assignment
        
4. Compute View Item Metrics
SELECT
    test_assignment
    ,count(item_id)                                         AS items
    ,sum(view_binary)                                       AS items_with_views
    ,sum(view_binary_30d)                                   AS items_with_recent_views
    ,avg(view_events)                                       AS avg_recent_views
    ,sum(view_binary_30d)*100/count(item_id)                AS percentage_recent_views_per_assigned_item
  FROM 
    (SELECT
        assignments.item_id
        ,assignments.test_number
        ,assignments.test_assignment
        ,MAX(CASE 
              WHEN views.event_time > assignments.test_start_date
              THEN 1
              ELSE 0 
              END)                                          AS view_binary
        ,MAX(CASE 
              WHEN (views.event_time > assignments.test_start_date AND date_part('day',views.event_time - assignments.test_start_date) <= 30)
              THEN 1
              ELSE 0 
              END)                                          AS view_binary_30d
        ,COUNT(DISTINCT CASE 
              WHEN views.event_time > assignments.test_start_date
              THEN views.event_time 
              ELSE NULL 
              END)                                          AS view_events
      FROM
        ( SELECT 
            *
          FROM 
            dsv1069.final_assignments 
          WHERE
            test_number = 'item_test_2'
        )                                                    AS assignments
      LEFT JOIN
        ( SELECT 
            event_time
            ,max( CASE 
                  WHEN parameter_name = 'item_id'
                  THEN CAST(parameter_value AS INT)
                  ELSE NULL 
                  END)                                      AS item_id  
          FROM 
            dsv1069.events 
          WHERE
            event_name = 'view_item'
          GROUP BY
            event_time
        )                                                    AS views
        ON assignments.item_id = views.item_id
      GROUP BY
        assignments.item_id
        ,assignments.test_number
        ,assignments.test_assignment
    )                                                         AS item_level
  GROUP BY
    test_assignment
        
