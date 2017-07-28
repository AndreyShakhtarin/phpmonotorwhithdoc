All control
--------------------
Error reporting
-------------
> error reporting: 
           
    bootstrup.php:
    48: // check for a debug var
    49: if(!defined('PSM_DEBUG')) {
    50: define('PSM_DEBUG', true);
        
-----------
> chekc install:
        
    bootsrap.php
    // sanity check!
    70: if(!defined('PSM_INSTALL') || !PSM_INSTALL) {
    	if($db->getDbHost() === null) {
    		// no config file has been loaded, redirect the user to the install
    		header('Location: install.php');
    
    		die();
    	}
    	// config file has been loaded, check if we have a connection
    	if(!$db->status()) {
    		die('Unable to establish database connection...');
    	}
    	// attempt to load configuration from database
    	if(!psm_load_conf()) {
    		// unable to load from config table
    		header('Location: install.php');
    		die();
    	}
    	// config load OK, make sure database version is up to date
    	$installer = new \psm\Util\Install\Installer($db);
    	if($installer->isUpgradeRequired()) {
    		die('Your database is for an older version and requires an upgrade, <a href="install.php">please click here</a> to update your database to the latest version.');
    	}
    93:}   
 --------------------- 
Set username password and username
---

>set user for login 

    psm/Service/User.php
    141: public function getUserByUsername($user_name) {
                // database query, getting all the info of the selected user
        //        require __DIR__ . '/../../../config.php';
        //		$query_user = $this->db_connection->prepare('SELECT * FROM '.PSM_DB_PREFIX.'users WHERE user_name = :user_name');
        //		$query_user->bindValue(':user_name', $user_name, \PDO::PARAM_STR);
        //		$query_user->execute();
                // get result row (as an object)
                $user['name'] = PSM_LOGIN_USERNAME;
                $user['pass'] = PSM_LOGIN_PASSWORD;
                return $user;
            }
        
 -----------------

Check tables
 -----------
>bootstrap.php

    83: if(!psm_load_conf()) {
    		// unable to load from config table
    		header('Location: install.php');
    		die();
    87:	}
    
>includes/function.inc.php

    158:function psm_load_conf() {
            global $db;
        
            $GLOBALS['sm_config'] = array();
        
            if(!defined('PSM_DB_PREFIX') || !$db->status()) {
                return false;
            }
            if(!$db->ifTableExists(PSM_DB_PREFIX.'config')) {
                return false;
            }
            $config_db = $db->select(PSM_DB_PREFIX . 'config', null, array('key', 'value'));
        
            if(is_array($config_db) && !empty($config_db)) {
                foreach($config_db as $setting) {
                    $GLOBALS['sm_config'][$setting['key']] = $setting['value'];
                }
                return true;
            } else {
                return false;
            }
    179: }
------------------

  Create tables 
 ------------------
 >psm/Utils/install/Installer.php
 
        173: protected function installTables() {
                ....
        265: }
 ---------------------    
 
 Controllers
 -----------------
 DefaultController indexAction()
 
 >src/psm/Module/Server/Controller/StatusController.php
 
    48: protected function executeIndex()
---------------------------
Define controller
----------------------------------
 
 >set module and service
 
  +
 
    src/config/services.xml: set module
    
        <parameters>
            <parameter key="config.theme" type="constant">PSM_THEME</parameter>
            <parameter key="modules" type="collection">
                <parameter>module.config</parameter>
                    ...........
                </parameter>
 
  + set service 
        
        .........
        <services>
        	<!--MODULES start-->
        	<service id="module.config" class="psm\Module\Config\ConfigModule" />
       		<service id="module.error" class="psm\Module\Error\ErrorModule" />

> Create Module 
+ src/psm/Module/module_Name/module_nameModule.php

        namespace psm\Module\module_name;
        
        use psm\Module\ModuleInterface;
        use Symfony\Component\DependencyInjection\ContainerBuilder;
        
        class module_nameModule implements ModuleInterface {
        
        	public function load(ContainerBuilder $container) {
        
        	}
        
        	public function getControllers() {
        		return array(
        			'config' => __NAMESPACE__ . '\Controller\name_controllerController',
        			(base name controller nema of module)
        		);
        	}
        }
        
> Create Controller
+ src/psm/Module/module_name/Controller/name_controller
    
        namespace psm\Module\modul_name\Controller;
        use psm\Module\AbstractController;
        use psm\Service\Database;
        
        class name_controller extends AbstractController {
            function __construct(Database $db, \Twig_Environment $twig) {
            		parent::__construct($db, $twig);
            
            		$this->setMinUserLevelRequired(PSM_USER_ADMIN);
            		$this->setCSRFKey('config');
                    //set action for route
            		$this->setActions(array(
            			'index', 'save',
            		), 'index');
            	}
            	
            	//IndexAction for route define this controller
            	protected function executeIndex() {
            	    //set header
                	$this->twig->addGlobal('subtitle', psm_get_lang('menu', 'config'));
                	// variable for translate
                	$tpl_data = $this->getLabels();
                	//your logic for controller
                	..............
                	//redering for template
                	return $this->twig->render('module/config/config.tpl.html', $tpl_data);
                	}
--------                	
Database
-------
+ src/psm/Service/Database.php

    class Database {
    
    	/**
    	 * DB hostname
    	 * @var string $db_host
    	 */
    	protected $db_host;
    
    	/**
    	 * DB port
    	 * @var string $db_port
    	 */
    	protected $db_port = 3306;
    
    	/**
    	 * DB name
    	 * @var string $db_name
    	 */
    	protected $db_name;
    
    	/**
    	 * DB user password
    	 * @var string $db_pass
    	 */
    	protected $db_pass;
    
    	/**
    	 * DB username
    	 * @var string $db_user
    	 */
    	protected $db_user;
    
    	/**
    	 * PDOStatement of last query
    	 * @var \PDOStatement $last
    	 */
    	protected $last;
    
    	/**
    	 * Mysql db connection identifer
    	 * @var \PDO $pdo
    	 * @see pdo()
    	 */
    	protected $pdo;
    
    	/**
    	 * Connect status
    	 * @var boolean
    	 * @see connect()
    	 */
    	protected $status = false;
    	
    	function __construct($host = null, $user = null, $pass = null, $db = null, $port = null) {
        		if($host != null && $user != null && $pass !== null && $db != null) {
        			$this->db_host = $host;
        			$this->db_port = (!empty($port)) ? $port : 3306;
        			$this->db_name = $db;
        			$this->db_user = $user;
        			$this->db_pass = $pass;
        			$this->connect();
        		}
        	}
        	...........
        }
----------------
Base query 

+ queery($query, $fetch =true);

        public function query($query, $fetch = true) {
                // Execute query and process results
                try {
                    $this->last = $this->pdo()->query($query);
                } catch (\PDOException $e) {
                    $this->error($e);
                }
        
                if($fetch && $this->last != false) {
                    $cmd = strtolower(substr($query, 0, 6));
        
                    switch($cmd) {
                        case 'insert':
                            // insert query, return insert id
                            $result = $this->getLastInsertedId();
                            break;
                        case 'update':
                        case 'delete':
                            // update/delete, returns rowCount
                            $result = $this->getNumRows();
                            break;
                        default:
                            $result = $this->last->fetchAll(\PDO::FETCH_ASSOC);
                            break;
                    }
                } else {
                    $result = $this->last;
                }
                return $result;
            }
            
+ execute($query, $parameters, $fetch = true)

        public function execute($query, $parameters, $fetch = true) {
        		try {
        			$this->last = $this->pdo()->prepare($query);
        			$this->last->execute($parameters);
        		} catch (\PDOException $e) {
        			$this->error($e);
        		}
        
        		if($fetch && $this->last != false) {
        			$result = $this->last->fetchAll(\PDO::FETCH_ASSOC);
        		} else {
        			$result = $this->last;
        		}
        		return $result;
        	}
        	
+ select($table, $where = null, $fields = null, $limit = '', $orderby = null, $direction = 'ASC')

        public function select($table, $where = null, $fields = null, $limit = '', $orderby = null, $direction = 'ASC'){
        		// build query
        		$query_parts = array();
        		$query_parts[] = 'SELECT SQL_CALC_FOUND_ROWS';
        
        		// Fields
        		if ($fields !== null && !empty($fields)) {
        			$query_parts[] = "`".implode('`,`', $fields)."`";
        		} else {
        			$query_parts[] = ' * ';
        		}
        
        		// From
        		$query_parts[] = "FROM `{$table}`";
        
        		// Where clause
        		$query_parts[] = $this->buildSQLClauseWhere($table, $where);
        
        		// Order by
        		if($orderby) {
        			$query_parts[] = $this->buildSQLClauseOrderBy($orderby, $direction);
        		}
        
        		// Limit
        		if ($limit != '') {
        			$query_parts[] = 'LIMIT ' . $limit;
        		}
        
        		$query = implode(' ', $query_parts);
        
        		return $this->query($query);
        	}
        	
+ selectRow($table, $where = null, $fields = null, $orderby = null, $direction = 'ASC')
        
        public function selectRow($table, $where = null, $fields = null, $orderby = null, $direction = 'ASC') {
        		$result = $this->select($table, $where, $fields, '1', $orderby, $direction);
        
        		if(isset($result[0])) {
        			$result = $result[0];
        		}
        
        		return $result;
        	}
        	
+ delete($table, $where = null)

        public function delete($table, $where = null){
        		$sql = 'DELETE FROM `'.$table.'` ' . $this->buildSQLClauseWhere($table, $where);
        
        		return $this->exec($sql);
        	}
        	
+ save($table, array $data, $where = null)

        public function save($table, array $data, $where = null) {
        		if ($where === null) {
        			// insert mode
        			$query = "INSERT INTO ";
        			$exec = false;
        		} else {
        			$query = "UPDATE ";
        			$exec = true;
        		}
        
        		$query .= "`{$table}` SET ";
        
        		foreach($data as $field => $value) {
        			if(is_null($value)) {
        				$value = 'NULL';
        			} else {
        				$value = $this->quote($value);
        			}
        			$query .= "`{$table}`.`{$field}`={$value}, ";
        		}
        
        		$query = substr($query, 0, -2) . ' ' . $this->buildSQLClauseWhere($table, $where);
        
        		if($exec) {
        			return $this->exec($query);
        		} else {
        			return $this->query($query);
        		}
        	}
        	
+ insertMultiple($table, array $data)

        public function insertMultiple($table, array $data) {
        		if(empty($data)) return false;
        
        		// prepare first part
        		$query = "INSERT INTO `{$table}` ";
        		$fields = array_keys($data[0]);
        		$query .= "(`".implode('`,`', $fields)."`) VALUES ";
        
        		// prepare all rows to be inserted with placeholders for vars (\?)
        		$q_part = array_fill(0, count($fields), '?');
        		$q_part = "(".implode(',', $q_part).")";
        
        		$q_part = array_fill(0, count($data), $q_part);
        		$query .= implode(',', $q_part);
        
        		$pst = $this->pdo()->prepare($query);
        
        		$i = 1;
        		foreach($data as $row) {
        			// make sure the fields of this row are identical to first row
        			$diff_keys = array_diff_key($fields, array_keys($row));
        
        			if(!empty($diff_keys)) {
        				continue;
        			}
        			foreach($fields as $field) {
        				$pst->bindParam($i++, $row[$field]);
        			}
        		}
        
        		try {
        			$this->last = $pst->execute();
        		} catch (\PDOException $e) {
        			$this->error($e);
        		}
        		return $this->last;
        	}
        	
+ buildSQLClauseWhere($table, $where = null)
        public function buildSQLClauseWhere($table, $where = null) {
        
        		$query = '';
        
        		if ($where !== null) {
        			if (is_array($where)) {
        				$query .= " WHERE ";
        
        				foreach($where as $field => $value) {
        					$query .= "`{$table}`.`{$field}`={$this->quote($value)} AND ";
        				}
        				$query = substr($query, 0, -5);
        			} else {
        				if (strpos($where, '=') === false) {
        					// no field given, use primary field
        					$primary = $this->getPrimary($table);
        					$query .= " WHERE `{$table}`.`{$primary}`={$this->quote($where)}";
        				} elseif (strpos(strtolower(trim($where)), 'where') === false) {
        					$query .= " WHERE {$where}";
        				} else {
        					$query .= ' '.$where;
        				}
        			}
        		}
        		return $query;
        	}
        	
+ buildSQLClauseOrderBy($order_by, $direction)

        public function buildSQLClauseOrderBy($order_by, $direction) {
        		$query = '';
        
        		if ($order_by !== null) {
        			if (is_array($order_by)) {
        				$query .= " ORDER BY ";
        
        				foreach($order_by as $field) {
        					$query .= "`{$field}`, ";
        				}
        				// remove trailing ", "
        				$query = substr($query, 0, -2);
        			} else {
        				if (strpos(strtolower(trim($order_by)), 'order by') === false) {
        					$query .= " ORDER BY {$order_by}";
        				} else {
        					$query .= ' '.$order_by;
        				}
        			}
        		}
        		if(strlen($query) > 0) {
        			// check if "ASC" or "DESC" is already in the order by clause
        			if(strpos(strtolower(trim($query)), 'asc') === false && strpos(strtolower(trim($query)), 'desc') === false) {
        				$query .= ' '.$direction;
        			}
        		}
        
        		return $query;
        	}
        	
end other for operation for connect
