# wordpress_hook_to_sort_new_column
<br/>

## The problem :
####  I want to add new custom sortable column to Users Admin panel in WordPress
<br/>

### Solution :
#### Suppose your new column will be named as type, and the meta_key in (user_meta) db is 'type'.
#### Add these lines to you functions.php
<br/>

```
// add additional columns to the users.php admin page
function modify_user_columns($column) {
    $column = array(
    "cb" => "",
    "username" => "Username",
    "name" => "Name",
    "email" => "E-mail",
    "role" => "Role",
    "posts" => "Posts",
    "type" => "Type"  //the new column
    );
    return $column;
}
add_filter('manage_users_columns','modify_user_columns');


//add content to your new custom column
function modify_user_column_content($val,$column_name,$user_id) {
    switch ($column_name) {
      case 'type':
      $meta = get_the_author_meta('type', $user_id);  // get the user meta
      return $meta ;
      break;

      default:
    }
    return $return;
}
add_filter('manage_users_custom_column', 'modify_user_column_content', 10, 3);

//make the new column sortable
function user_sortable_columns( $columns ) {
  $columns['type'] = 'type';
  return $columns;
}
add_filter( 'manage_users_sortable_columns', 'user_sortable_columns' );

//set instructions on how to sort the new column
if(is_admin()) { //prolly not necessary, but I do want to be sure this only runs within the admin
  add_action('pre_user_query', 'my_user_query');
}
function my_user_query($userquery){
  if('type'==$userquery->query_vars['orderby']) {//check if type is the column being sorted
    global $wpdb;
    $userquery->query_from .= " LEFT OUTER JOIN $wpdb->usermeta AS alias ON ($wpdb->users.ID = alias.user_id) ";//note use of alias
    $userquery->query_where .= " AND alias.meta_key = 'type' ";//which meta are we sorting with?
    $userquery->query_orderby = " ORDER BY alias.meta_value ".($userquery->query_vars["order"] == "ASC" ? "asc " : "desc ");//set sort order
  }
}
```

That's it !
