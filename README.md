# teste-sqlite

import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';

class DBHelper {
  static final String _nomeDB = "controleFutebol.db";
  static final int _versaoDB = 1;

  static final String tableJogadores = "jogadores";
  static final String jogadorId = "jogador_id";
  static final String jogadorNome = "nome";
  static final String jogadorSituacao = "situacao";
  static final String jogadorSaldo = "saldo";

  static final String tablePartidas = "partidas";
  static final String partidaId = "partida_id";
  static final String partidaData = "data";
  static final String partidaDescricao = "descricao";
  static final String partidaSituacao = "situacao";

  static final String tableDespesas = "despesas";
  static final String despesaId = "despesa_id";
  static final String despesaDescricao = "descricao";
  static final String despesaValor = "valor";
  static final String despesaDataHora = "data_hora";

  // Aplicação do padrão Singleton na classe.
  DBHelper._privateConstructor();
  static final DBHelper instance = DBHelper._privateConstructor();

  // Configurar a intância única da classe. 
  // Abre a base de dados (e cria quando ainda não existir).
  static Database _database;
  Future<Database> get database async {
    if (_database != null) return _database;

    _database = await _initDatabase();
    return _database;
  }

  Future<Database> _initDatabase() async {
    
    String caminhoDoBanco = await getDatabasesPath();
    String _banco = _nomeDB;
    String path = join(caminhoDoBanco, _banco);

    return await openDatabase(
      path,
      version: _versaoDB,
      onCreate: _criarBanco,
    );
  }

  Future<void> _criarBanco(Database db, int novaVersao) async {

    List<String> queryes = [
      "CREATE TABLE $tableJogadores ($jogadorId INTEGER PRIMARY KEY, $jogadorNome TEXT, $jogadorSituacao TEXT, $jogadorSaldo FLOAT);",
      "CREATE TABLE $tablePartidas ($partidaId INTEGER PRIMARY KEY, $partidaDescricao TEXT, $partidaData TEXT, $partidaSituacao TEXT);",
      "CREATE TABLE $tableDespesas ($despesaId INTEGER PRIMARY KEY, $despesaDescricao TEXT, $despesaValor FLOAT, $despesaDataHora TEXT);",
    ];

    for (String query in queryes) {
      await db.execute(query);
    }
  }
}
PageJogador.dart
import 'package:contole_futebol/src/data/DBHelper/repositories/RepositoryJogador.dart';
import 'package:contole_futebol/src/models/jogador.dart';
import 'package:flutter/material.dart';

class Jogadores extends StatefulWidget {
  @override
  _JogadoresState createState() => _JogadoresState();
}

class _JogadoresState extends State<Jogadores> {
  RepositoryJogador repository = RepositoryJogador();

  List<Jogador> jogadores = [];

  @override
  Widget build(BuildContext context) {
    _recuperarJogadores();
    return Scaffold(
      appBar: _appBar(),
      floatingActionButton: _floatingActionButton(),
      body: _body(),
    );
  }

  FloatingActionButton _floatingActionButton() {
    return FloatingActionButton(

      child: Icon(Icons.person_add),
      onPressed: () {
        print("Adicionar novo jogador");
        _adicionarJogador();
      },
    );
  }

  AppBar _appBar() {
    return AppBar(
      title: Text("Jogadores"),
    );
  }

  _body() {
    return Container(
      child: ListView.builder(
        itemCount: jogadores.length,
        itemBuilder: (BuildContext context, int index) {
          Jogador _jogador = jogadores[index];
          return Container(
            decoration: BoxDecoration(
              border: Border(
                bottom: BorderSide(
                  width: 0.5,
                  color: Colors.amber.withOpacity(0.6),
                ),
              ),
            ),
            child: ListTile(
              title: Text(_jogador.nome),
              subtitle: Text("Valor devido"),
              trailing: CircleAvatar(
                backgroundColor: _cor(_jogador.situacao),
              ),
              onTap: () {},
            ),
          );
        },
      ),
    );
  }

  Color _cor(Situacao situacao) {
    Color _cor;
    switch (situacao) {
      case Situacao.ok:
        _cor = Colors.greenAccent.withOpacity(0.5);
        break;
      case Situacao.comCredito:
        _cor = Colors.blueAccent.withOpacity(0.5);
        break;
      case Situacao.devedor:
        _cor = Colors.redAccent.withOpacity(0.5);
        break;
    }

    return _cor;
  }

  Future _adicionarJogador() async {
    setState(() {
      Jogador j = Jogador(
        id: 1,
        nome: "Leonardo Henrique Paim",
        situacao: Situacao.ok,
      );
      repository.insertJogador(j);
    });
  }

  _recuperarJogadores() async {
    var lista = await repository.selectJogador();
    setState(() {
      jogadores = lista;
    });
  }
}
RepositoryDespesa.dart
import 'package:contole_futebol/src/data/DBHelper/DBHelper.dart';
import 'package:contole_futebol/src/models/despesa.dart';
import 'package:sqflite/sqflite.dart';

class RepositoryDespesa {
  Future<void> insertDespesa(Despesa despesa) async {
    Database db = await DBHelper.instance.database;

    Map<String, dynamic> row = {
      DBHelper.despesaDescricao: despesa.descricao,
      DBHelper.despesaValor: despesa.valor,
      DBHelper.despesaDataHora: despesa.dataHora.toString(),
    };

    await db.insert(DBHelper.tableDespesas, row);
  }

  Future<List<Despesa>> selectDespesas({int id}) async{
    Database db = await DBHelper.instance.database;

    List<Despesa> retorno = [];
    List<Map> despesas = await db.rawQuery("SELECT * FROM ${DBHelper.tableDespesas}");

    despesas.forEach((despesa){
      retorno.add(
        Despesa(
          id: despesa[DBHelper.despesaId],
          descricao: despesa[DBHelper.despesaDescricao],
          valor: despesa[DBHelper.despesaValor],
          dataHora: despesa[DBHelper.despesaDataHora],
        )
      );
    });

    return retorno;
  }
}
RepositoryJogador.dart
import 'package:contole_futebol/src/data/DBHelper/DBHelper.dart';
import 'package:contole_futebol/src/models/jogador.dart';
import 'package:sqflite/sqflite.dart';

class RepositoryJogador {
  insertJogador(Jogador jogador) async {
    Database db = await DBHelper.instance.database;

    Map<String, dynamic> row = {
      DBHelper.jogadorNome: jogador.nome,
      DBHelper.jogadorSituacao: jogador.situacao.toString(),
    };
    await db.insert(DBHelper.tableJogadores, row);
  }

  Future<List<Jogador>> selectJogador() async {
    Database db = await DBHelper.instance.database;
    List<Jogador> retorno = [];
    List<Map> jogadores =
        await db.rawQuery("SELECT * FROM ${DBHelper.tableJogadores}");

    jogadores.forEach((jogador) {
      retorno.add(
        Jogador(
          id: jogador[DBHelper.jogadorId],
          nome: jogador[DBHelper.jogadorNome],
          //situacao: jogador[DBHelper.jogadorSituacao],
        ),
      );
    });

    return retorno;
  }
}
RepositoryPartida.dart
import 'package:contole_futebol/src/data/DBHelper/DBHelper.dart';
import 'package:contole_futebol/src/models/partida.dart';
import 'package:sqflite/sqflite.dart';

class RepositoryPartida {
  insertPartida(Partida partida) async {
    Database db = await DBHelper.instance.database;

    Map<String, dynamic> row = {
      DBHelper.partidaDescricao: partida.descricao,
      DBHelper.partidaData: partida.data.toString(),      
      DBHelper.partidaSituacao: partida.situacao.toString(),
    };
    await db.insert(DBHelper.tablePartidas, row);
  }  

  Future<List<Partida>> selectPartida() async {
    Database db = await DBHelper.instance.database;
    List<Partida> retorno = [];
    List<Map> partidas =
        await db.rawQuery("SELECT * FROM ${DBHelper.tablePartidas}");

    partidas.forEach((partida) {
      retorno.add(
        Partida(
          id: partida[DBHelper.partidaId],
          descricao: partida[DBHelper.partidaDescricao], 
          data: "",
          situacao: SituacaoPartida.finalizada,
        ),
      );
    });

    return retorno;
  }
}
