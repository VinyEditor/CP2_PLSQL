RM: 97210
Nome: Vinicius Cruzeiro Barbosa
FIAP
Database Application e Data Science
CP2


CREATE TABLE Pedido (
    Id_ped INT PRIMARY KEY,
    dt_ped DATE NOT NULL,
    total_ped DECIMAL(10, 2) NOT NULL
);

CREATE TABLE Itemped (
    Id_ped INT NOT NULL,
    Id_mus INT NOT  NULL,
    PRIMARY KEY (Id_ped, Id_mus),
    FOREIGN KEY (Id_ped) REFERENCES Pedido(Id_ped) ON DELETE CASCADE,
    FOREIGN KEY (Id_mus) REFERENCES Musica(Id_mus) ON DELETE CASCADE
);

CREATE TABLE Musica (
    Id_mus INT PRIMARY KEY,
    Tit_mus VARCHAR(255) NOT NULL,
    Valor_mus DECIMAL(10, 2) NOT NULL
);

INSERT ALL
  INTO Pedido VALUES (001, '25-jun-2015', 50.00);
  INTO Pedido VALUES (002, '01-oct-2019', 65.00);
  INTO Pedido VALUES (003, '13-apr-2023', 25.00);
  INTO Pedido VALUES (004, '08-dez-2007', 10.00);
  INTO Pedido VALUES (005, '30-aug-2021', 45.00);
SELECT * FROM Pedido;
commit;

INSERT ALL
  INTO Musica VALUES (001, 'Livin' On A Prayer', 40.00);
  INTO Musica VALUES (002, 'Back in Black', 15.00);
  INTO Musica VALUES (003, 'Master Of Puppets', 15.00);
  INTO Musica VALUES (005, 'Seven Nation Army',20.00 );
  INTO Musica VALUES (006, 'Smoke On The Water', 10.00);
  INTO Musica VALUES (007, 'Crazy Train', 30.00);
  INTO Musica VALUES (008, 'Mary on a Cross', 5.00);
  INTO Musica VALUES (009, 'Best of you', 5.00);
  INTO Musica VALUES (010, 'Fuel', 10.00);
SELECT * FROM Musica;
commit;

CREATE FUNCTION calculate_order_total(order_id INT)
RETURNS DECIMAL(10, 2)
BEGIN
    DECLARE order_total DECIMAL(10, 2);
    SELECT SUM(m.Valor_mus) INTO order_total
    FROM Pedido p
    JOIN Itemped i ON p.Id_ped = i.Id_ped
    JOIN Musica m ON i.Id_mus = m.Id_mus
    WHERE p.Id_ped = order_id;
    IF order_total IS NULL THEN
        SET order_total = 0;
    END IF;
    RETURN order_total;
END;

CREATE PROCEDURE display_order_details(order_id INT)
BEGIN
    SELECT p.Id_ped, p.dt_ped, p.total_ped, m.Tit_mus, m.Valor_mus
    FROM Pedido p
    JOIN Itemped i ON p.Id_ped = i.Id_ped
    JOIN Musica m ON i.Id_mus = m.Id_mus
    WHERE p.Id_ped = order_id;
    IF ROW_COUNT() = 0 THEN
        SELECT 'Nenhum pedido encontrado com o ID fornecido.' AS message;
    END IF;
END;

CREATE TRIGGER control_music_price_update
BEFORE UPDATE 
OF Valor_mus 
ON Musica
FOR EACH ROW
BEGIN
    IF NEW.Valor_mus < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'O preço da musica não pode ser negativo.';
    END IF;
    IF NEW.Valor_mus > 100 THEN
        SET NEW.Valor_mus = 100;
    END IF;
END;