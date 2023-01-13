# Motoko-Bootcamp-
escriba Id. de lote = nacional;
escriba ChunkId = nacional;
escriba Clave = texto;
escriba Tiempo = int;

escriba CreateAssetArguments = registro {
  llave llave;
  tipo_de_contenido: texto;
  max_age: opt nat64;
  encabezados: opt vec HeaderField;
  habilitar_aliasing: opt bool;
  allow_raw_access: opt bool;
};

// Agregar o cambiar contenido para un activo, por codificación de contenido
tipo SetAssetContentArguments = registro {
  llave llave;
  codificación_contenido: texto;
  chunk_ids: vec ChunkId;
  sha256: optar gota;
};

// Quitar el contenido de un activo, por codificación de contenido
escriba UnsetAssetContentArguments = registro {
  llave llave;
  codificación_contenido: texto;
};

// Eliminar un activo
escriba DeleteAssetArguments = registro {
  llave llave;
};

// Restablecer todo
escriba ClearArguments = registro {};

tipo BatchOperationKind = variante {
  CreateAsset: CreateAssetArguments;
  SetAssetContent: SetAssetContentArguments;

  UnsetAssetContent: UnsetAssetContentArguments;
  DeleteAsset: DeleteAssetArguments;

  Borrar: Argumentos claros;
};

escriba HeaderField = registro { texto; texto; };

escriba HttpRequest = registro {
  método: texto;
  URL: texto;
  encabezados: vec HeaderField;
  cuerpo: gota;
};

escriba HttpResponse = registro {
  código_estado: nat16;
  encabezados: vec HeaderField;
  cuerpo: gota;
  estrategia_streaming: opta por StreamingStrategy;
};

escriba StreamingCallbackHttpResponse = registro {
  cuerpo: gota;
  token: opta por StreamingCallbackToken;
};

escriba StreamingCallbackToken = registro {
  llave llave;
  codificación_contenido: texto;
  índice: natural;
  sha256: optar gota;
};

escriba StreamingStrategy = variante {
  Devolución de llamada: grabar {
    devolución de llamada: func (StreamingCallbackToken) -> (opt StreamingCallbackHttpResponse) consulta;
    token: StreamingCallbackToken;
  };
};

tipo SetAssetPropertiesArguments = registro {
  llave llave;
  max_age: opt opt ​​nat64;
  encabezados: opt opt ​​vec HeaderField;
  allow_raw_access: opt opt ​​bool;
};

Servicio: {
  obtener: (registro {
    llave llave;
    accept_encodings: texto vec;
  }) -> (registro {
    contenido: gota; // puede ser la totalidad del contenido, o solo el índice de fragmento 0
    tipo_de_contenido: texto;
    codificación_contenido: texto;
    sha256: optar gota; // sha256 de la codificación de activos completa, calculada por dfx y pasada en SetAssetContentArguments
    longitud_total: natural; // todos los fragmentos excepto el último tienen tamaño == content.size()
  }) consulta;

  // si get() devolvió fragmentos > 1, llame a esto para recuperarlos.
  // los fragmentos pueden dividirse o no en los mismos límites que se presentan en create_chunk().
  get_chunk: (registro {
    llave llave;
    codificación_contenido: texto;
    índice: nacional;
    sha256: optar gota; // sha256 de la codificación de activos completa, calculada por dfx y pasada en SetAssetContentArguments
  }) -> (registro { contenido: blob }) consulta;

  lista: (registro {}) -> (registro vec {
    llave llave;
    tipo_de_contenido: texto;
    codificaciones: registro vec {
      codificación_contenido: texto;
      sha256: optar gota; // sha256 de la codificación de activos completa, calculada por dfx y pasada en SetAssetContentArguments
      longitud: natural; // Tamaño del blob de esta codificación. Calculado al cargar activos.
      modificado: Tiempo;
    };
  }) consulta;

  árbol_certificado: (registro {}) -> (registro {
    certificado: gota;
    árbol: gota;
  }) consulta;

  create_batch : (registro {}) -> (registro { batch_id: BatchId });

  create_chunk: (registro { batch_id: BatchId; contenido: blob }) -> (registro { chunk_id: ChunkId });

  // Realizar todas las operaciones con éxito, o rechazar
  commit_batch: (registro { batch_id: BatchId; operaciones: vec BatchOperationKind }) -> ();

  create_asset: (CreateAssetArguments) -> ();
  set_asset_content: (SetAssetContentArguments) -> ();
  unset_asset_content: (UnsetAssetContentArguments) -> ();

  delete_asset: (DeleteAssetArguments) -> ();

  borrar: (Borrar Argumentos) -> ();

  // Llamada única para crear un activo con contenido para una codificación de contenido único que
  // cabe dentro del límite de entrada de mensajes.
  tienda: (registro {
    llave llave;
    tipo_de_contenido: texto;
    codificación_contenido: texto;
    contenido: gota;
    sha256: optar gota
  }) -> ();

  http_request: (solicitud: HttpRequest) -> (HttpResponse) consulta;
  http_request_streaming_callback: (token: StreamingCallbackToken) -> (opt StreamingCallbackHttpResponse) consulta;

  autorizar: (principal) -> ();
  desautorizar: (principal) -> ();
  list_authorized: () -> (vec principal) consulta;

  get_asset_properties : (clave: Clave) -> (registro {
    max_age: opt nat64;
    encabezados: opt vec HeaderField;
    allow_raw_access: opt bool; } ) consulta;
  set_asset_properties: (Argumentos de SetAssetProperties) -> ();
}
