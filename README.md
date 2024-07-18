CREATE OR REPLACE PROCEDURE process_indicators AS
  CURSOR cur IS
    SELECT t1.*, t2.t2_flex, t2.t2_sap, t2.t2_tdb
    FROM table1 t1
    JOIN table2 t2 ON t1.id = t2.id; -- Modify the join condition as per your schema

  line_ref CONSTANT VARCHAR2(20) := 'line_ref';
  else_line_ref CONSTANT VARCHAR2(20) := 'else_line_ref';

BEGIN
  FOR rec IN cur LOOP
    DECLARE
      result VARCHAR2(20);
    BEGIN
      result :=
        CASE rec.flexind
          WHEN 'E' THEN
            CASE 
              WHEN INSTR(rec.t2_flex, rec.t1_flex) > 0 THEN
                CASE 
                  WHEN rec.sapind = 'E' AND rec.t1_sap = rec.t2_sap THEN
                    CASE 
                      WHEN rec.tdbind = 'E' AND rec.t1_tdb = rec.t2_tdb THEN line_ref
                      WHEN rec.tdbind = 'P' THEN line_ref
                      ELSE else_line_ref
                    END
                  WHEN rec.sapind = 'P' THEN
                    CASE 
                      WHEN rec.tdbind = 'E' AND rec.t1_tdb = rec.t2_tdb THEN line_ref
                      WHEN rec.tdbind = 'P' THEN line_ref
                      ELSE else_line_ref
                    END
                  ELSE
                    else_line_ref
                END
              ELSE
                else_line_ref
            END
          WHEN 'P' THEN
            CASE 
              WHEN rec.sapind = 'E' AND rec.t1_sap = rec.t2_sap THEN
                CASE 
                  WHEN rec.tdbind = 'E' AND rec.t1_tdb = rec.t2_tdb THEN line_ref
                  WHEN rec.tdbind = 'P' THEN line_ref
                  ELSE else_line_ref
                END
              WHEN rec.sapind = 'P' THEN
                CASE 
                  WHEN rec.tdbind = 'E' AND rec.t1_tdb = rec.t2_tdb THEN line_ref
                  WHEN rec.tdbind = 'P' THEN line_ref
                  ELSE else_line_ref
                END
              ELSE
                else_line_ref
            END
          WHEN 'NE' THEN
            CASE 
              WHEN rec.sapind = 'NE' THEN
                CASE 
                  WHEN rec.t1_sap != rec.t2_sap THEN
                    CASE 
                      WHEN rec.tdbind = 'E' AND rec.t1_tdb != rec.t2_tdb THEN line_ref
                      WHEN rec.tdbind = 'P' THEN line_ref
                      ELSE else_line_ref
                    END
                  ELSE
                    else_line_ref
                END
              ELSE
                else_line_ref
            END
          ELSE
            else_line_ref
        END;

      -- Output the result
      DBMS_OUTPUT.PUT_LINE('ID ' || rec.id || ': ' || result);

    END;
  END LOOP;
END;
/
