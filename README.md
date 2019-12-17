# PHP to CSV

[![aledc.com](https://github.com/aledc7/Scrum-Certification/blob/master/recursos/aledc.com.svg)](https://aledc.com)
[![ingenea.com.ar](https://github.com/aledc7/Scrum-Certification/blob/master/recursos/ingenea.svg)](http://ingenea.com.ar)
[![License](https://github.com/aledc7/Scrum-Certification/blob/master/recursos/mit-license.svg)](https://aledc.com)
[![GitHub release](https://github.com/aledc7/Scrum-Certification/blob/master/recursos/release.svg)](https://aledc.com)
[![Dependencies](https://github.com/aledc7/Scrum-Certification/blob/master/recursos/dependencias-none.svg)](https://aledc.com)


Este repositorio muestra como convertir una __consulta SQL__ en un archivo  __.CSV__ . 
Luego tambien se muestra como evitar que el archivo csv quede con notación cientifica __E+15__  lo cual confunde mucho.


```php
<?php 

function exportxlsdataCSV() {
        // <editor-fold defaultstate="collapsed" desc="INCLUDE">
        $database = & $this->locator->get('database');
        $request = & $this->locator->get('request');
        $session = & $this->locator->get('session');
        $user = & $this->locator->get('user');
        $common = & $this->locator->get('common');
        // </editor-fold>

        $tabla = 'vw_csv_ope_registrosdellamadas';
        set_time_limit(0);

        try {
            $msg = '';

            $TABLE_NAME = $tabla;

            // <editor-fold defaultstate="collapsed" desc="ENCABEZADO">
            $sql = "SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name = '" . $TABLE_NAME . "' AND table_schema = '" . DB_NAME . "'  ";
            $list_colum_name = $database->getRows($sql);
            $cant_columnas = count($list_colum_name);
                 
            $nombre_de_las_columnas = array();
            for ($i = 0; $i < $cant_columnas ; $i++) {
                    $name = $list_colum_name[$i][COLUMN_NAME];
                    $nombre_de_las_columnas[] = $name;
            }
            // </editor-fold>
            
            // <editor-fold defaultstate="collapsed" desc="FILTRO Y CONSULTA">

            $sql = "SELECT * FROM vw_csv_ope_registrosdellamadas  ";
            $conca = " WHERE ";

        $fecha_d = '';
        if ($session->get('operegistrosdellamadas.desde') != '') {
            $dated = explode('/', $session->get('operegistrosdellamadas.desde'));
            $fecha_d = $dated[2] . '-' . $dated[1] . '-' . $dated[0];
        }
        if ($fecha_d != '') {
            $sql .= $conca . " FECHA_FILTRO >= '" . $fecha_d . "'";
            $conca = " AND ";
        }

        $fecha_h = '';
        if ($session->get('operegistrosdellamadas.hasta') != '') {
            $dated = explode('/', $session->get('operegistrosdellamadas.hasta'));
            $fecha_h = $dated[2] . '-' . $dated[1] . '-' . $dated[0];
        }
        if ($fecha_h != '') {
            $sql .= $conca . " FECHA_FILTRO <= '" . $fecha_h . "'";
            $conca = " AND ";
        }

        $sql .= " ORDER BY REGISTRO ASC";
        
           $results = $database->getRows($sql);
                      
        // </editor-fold>
        
            // <editor-fold defaultstate="collapsed" desc="FILTRO Y CONSULTA">
             set_time_limit(0);
                
                //LE METO EN ELCABEZADO
                array_unshift($results, $nombre_de_las_columnas);
                
            // </editor-fold>
            
            // <editor-fold defaultstate="collapsed" desc="TABLA SELECCIONADA A CSV">
            //cabeceras para descarga
            header('Content-Type: application/octet-stream');
            header("Content-Transfer-Encoding: Binary"); 
            header("Content-disposition: attachment; filename=\"Reporte RL.csv\""); 

            //preparar el wrapper de salida
            $outputBuffer = fopen("php://output", 'w');
            
            //volcamos el contenido del array en formato csv
             
            // (ale dc) le meto comillas simples a cada campo 
             function lemetocomilla($value) {
                return "'$value'";
            }
            
            foreach($results as $val) {                                          
            // (ale dc) uso la funcion array_map y llamo a la funcion que cree arriba          
            fputcsv($outputBuffer, array_map(lemetocomilla, $val), ';');                               
            }
                                  
            //cerramos el wrapper
            fclose($outputBuffer);
            exit;

            // </editor-fold>
            
        } catch (Exception $e) {
            $msg .= date('c') . "\r\n \t CATCH:\t " . $e . " \r\n";
            $common->log_msg($msg, 'excel_log');
        }
    }



?>


````



