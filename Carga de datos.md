# Carga  y transformación de los datos en Power BI

## Importación de Datos

**Descripción:** En este paso, importamos los datos proporcionados para el proyecto de Airbnb desde archivos CSV. Los datos proporcionados constan de tres tablas: calendar, listings y reviews.

- La tabla calendar contiene las fechas en las que los alojamientos estuvieron disponibles o no disponibles.
- La tabla de listings contiene toda la información relacionada con el alojamiento.
- La tabla reviews incluye los comentarios que han recibido los alojamientos, así como la puntuación que los usuarios les han otorgado.

A continuación, se muestra un fragmento del código utilizado para importar la tabla calendar. El proceso se repite para las tablas listings y reviews; la única modificación necesaria es en la ubicación del origen de los datos. Al final del documento, se proporcionará el código completo para la limpieza de las tres tablas.

```M
let
    Origen = Csv.Document(File.Contents("C:\Users\Grethel\Documents\Henry\M5 - Data Analytics\Proyecto Integrador\2 Airbnb\calendar.csv"), [Delimiter=",", Columns=7, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Encabezados Promovidos" = Table.PromoteHeaders(Origen, [PromoteAllScalars=true]),
```

### Normalización del nombre de las columnas

Para hacer uniforme cómo se van a llamar las columnas, la primera letra de cada palabra va a ir con letra mayúscula y la separación de palabras va a ser con un guión bajo ( _ ).

```M
let
    NombresColumnas = Table.ColumnNames(#"Encabezados Promovidos"),
    NuevosNombres = List.Transform(NombresColumnas, each Text.Proper(_)),
    RenamedColumns = Table.RenameColumns(#"Encabezados Promovidos", List.Zip({NombresColumnas, NuevosNombres}))
in
    RenamedColumns

```

### Eliminación de Columnas Innecesarias

Hay columnas que no son necesarias para realizar este análisis, por lo que las eliminé para que las tablas sean más fáciles de leer y evitar confusiones debido a datos innecesarios.

```M
#"Columnas quitadas4" = Table.RemoveColumns(#"Columnas con nombre cambiado2",{"Smart_Location", "Country_Code", "Country",})
```


### Convertir a Mayúsculas Cada Palabra

Se utilizó la opción de convertir a Mayúsculas cada palabra para que los valores de las filas de las columnas, todas las palabras comiencen con mayúsculas y que quede uniforme con el nombre de las columnas.

```M
#"Poner En Mayúsculas Cada Palabra" = Table.TransformColumns(#"Borrar columnas irrelevantes", {
    {"Name", Text.Proper, type text},
    {"Description", Text.Proper, type text},
    {"Host_Name", Text.Proper, type text},
    {"Host_Response_Time", Text.Proper, type text}
})

```

### Tratamiento a datos faltantes

Las filas que no tengan ningún valor o que su valor sea nulo serán reemplazadas por "No Data".

```M
#"Valor reemplazado" = Table.ReplaceValue(#"Poner En Mayúsculas Cada Palabra", "", "No Data", Replacer.ReplaceValue, {"Host_Response_Time"}),
#"Valor reemplazado1" = Table.ReplaceValue(#"Valor reemplazado", "N/A", "No Data", Replacer.ReplaceText, {"Host_Response_Time"})

```


### Normalización del texto en filas

En las tablas originalmente vienen algunas columnas que contienen las categorías "f" y "t". "f" significa "Falso" y "t" significa "Verdadero". Por lo tanto, "f" y "t" serán reemplazados por valores más comprensibles para el usuario. Por ejemplo, en la columna "Host_Is_Superhost", "f" se reemplazará por "Host" y "t" por "SuperHost".

```M
#"Valor reemplazado3" = Table.ReplaceValue(#"Valores reemplazados con No Data", "t", "Superhost", Replacer.ReplaceText, {"Host_Is_Superhost"}),
#"Valor reemplazado4" = Table.ReplaceValue(#"Valor reemplazado3", "f", "Host", Replacer.ReplaceText, {"Host_Is_Superhost"})

```

### Limpieza de las columnas tipo texto

Se aplicó la transformación de datos para recortar y limpiar las columnas de tipo texto, con el objetivo de eliminar espacios en blanco, avances de línea o caracteres de control de texto.

```M
#"Texto limpio" = Table.TransformColumns(#"Columnas con nombre cambiado", {{"Comments", Text.Clean, type text}}),
#"Texto recortado" = Table.TransformColumns(#"Texto limpio", {{"Comments", Text.Trim, type text}})

```

### Valores atípicos

Se filtra la columna de precio para eliminar los valores que se encuentren a más de tres desviaciones estándar de la media.



```M
Media = List.Average(#"Columnas quitadas5"[Price]),
    DesviacionEstandar = List.StandardDeviation(#"Columnas quitadas5"[Price]),
    LimiteInferior = Media - 3 * DesviacionEstandar,
    LimiteSuperior = Media + 3 * DesviacionEstandar,
    FilasFiltradas = Table.SelectRows(#"Columnas quitadas5", each [Price] >= LimiteInferior and [Price] <= LimiteSuperior)
in
    FilasFiltradas
```


---

## Código de la transformación de las Tablas de Airbnb


En esta sección, se muestra el código de transformación utilizado para las tablas.

<details>
  <summary>Tabla calendar</summary>

```M
let
    Origen = Csv.Document(File.Contents("C:\Users\Grethel\Documents\Henry\M5 -Data Anlytics\Proyecto Integrador\2 AirBnB\calendar.csv"), [Delimiter=",", Columns=7, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Encabezados promovidos" = Table.PromoteHeaders(Origen, [PromoteAllScalars=true]),
    #"Tipo cambiado" = Table.TransformColumnTypes(#"Encabezados promovidos", {
        {"listing_id", Int64.Type},
        {"date", type date},
        {"available", type text},
        {"price", Currency.Type},
        {"adjusted_price", Currency.Type},
        {"minimum_nights", Int64.Type},
        {"maximum_nights", Int64.Type}
    }),
    NombresColumnas = Table.ColumnNames(#"Tipo cambiado"),
    NuevosNombres = List.Transform(NombresColumnas, each Text.Proper(_)),
    RenamedColumns = Table.RenameColumns(#"Tipo cambiado", List.Zip({NombresColumnas, NuevosNombres})),
    CambiarAvailable = Table.TransformColumns(RenamedColumns, {{"Available", each if _ = "f" then "NoAvailable" else if _ = "t" then "Available" else _}})
in
    CambiarAvailable
```
</details>
<details>
  <summary>Tabla listing</summary>

```M
let
    Origen = Csv.Document(File.Contents("C:\Users\Grethel\Documents\Henry\M5 -Data Anlytics\Proyecto Integrador\2 AirBnB\listings.csv"),[Delimiter=",", Columns=106, Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
    #"Encabezados promovidos" = Table.PromoteHeaders(Origen, [PromoteAllScalars=true]),
    #"Tipo cambiado" = Table.TransformColumnTypes(#"Encabezados promovidos",{{"id", Int64.Type}, {"listing_url", type text}, {"scrape_id", Int64.Type}, {"last_scraped", type date}, {"name", type text}, {"summary", type text}, {"space", type text}, {"description", type text}, {"experiences_offered", type text}, {"neighborhood_overview", type text}, {"notes", type text}, {"transit", type text}, {"access", type text}, {"interaction", type text}, {"house_rules", type text}, {"thumbnail_url", type text}, {"medium_url", type text}, {"picture_url", type text}, {"xl_picture_url", type text}, {"host_id", Int64.Type}, {"host_url", type text}, {"host_name", type text}, {"host_since", type date}, {"host_location", type text}, {"host_about", type text}, {"host_response_time", type text}, {"host_response_rate", type text}, {"host_acceptance_rate", type text}, {"host_is_superhost", type text}, {"host_thumbnail_url", type text}, {"host_picture_url", type text}, {"host_neighbourhood", type text}, {"host_listings_count", Int64.Type}, {"host_total_listings_count", Int64.Type}, {"host_verifications", type text}, {"host_has_profile_pic", type text}, {"host_identity_verified", type text}, {"street", type text}, {"neighbourhood", type text}, {"neighbourhood_cleansed", type text}, {"neighbourhood_group_cleansed", type text}, {"city", type text}, {"state", type text}, {"zipcode", type text}, {"market", type text}, {"smart_location", type text}, {"country_code", type text}, {"country", type text}, {"latitude", type number}, {"longitude", type number}, {"is_location_exact", type text}, {"property_type", type text}, {"room_type", type text}, {"accommodates", Int64.Type}, {"bathrooms", type number}, {"bedrooms", Int64.Type}, {"beds", Int64.Type}, {"bed_type", type text}, {"amenities", type text}, {"square_feet", Int64.Type}, {"price", Currency.Type}, {"weekly_price", Currency.Type}, {"monthly_price", Currency.Type}, {"security_deposit", Currency.Type}, {"cleaning_fee", Currency.Type}, {"guests_included", Int64.Type}, {"extra_people", Currency.Type}, {"minimum_nights", Int64.Type}, {"maximum_nights", Int64.Type}, {"minimum_minimum_nights", Int64.Type}, {"maximum_minimum_nights", Int64.Type}, {"minimum_maximum_nights", Int64.Type}, {"maximum_maximum_nights", Int64.Type}, {"minimum_nights_avg_ntm", type number}, {"maximum_nights_avg_ntm", Int64.Type}, {"calendar_updated", type text}, {"has_availability", type text}, {"availability_30", Int64.Type}, {"availability_60", Int64.Type}, {"availability_90", Int64.Type}, {"availability_365", Int64.Type}, {"calendar_last_scraped", type date}, {"number_of_reviews", Int64.Type}, {"number_of_reviews_ltm", Int64.Type}, {"first_review", type date}, {"last_review", type date}, {"review_scores_rating", Int64.Type}, {"review_scores_accuracy", Int64.Type}, {"review_scores_cleanliness", Int64.Type}, {"review_scores_checkin", Int64.Type}, {"review_scores_communication", Int64.Type}, {"review_scores_location", Int64.Type}, {"review_scores_value", Int64.Type}, {"requires_license", type text}, {"license", type text}, {"jurisdiction_names", type text}, {"instant_bookable", type text}, {"is_business_travel_ready", type text}, {"cancellation_policy", type text}, {"require_guest_profile_picture", type text}, {"require_guest_phone_verification", type text}, {"calculated_host_listings_count", Int64.Type}, {"calculated_host_listings_count_entire_homes", Int64.Type}, {"calculated_host_listings_count_private_rooms", Int64.Type}, {"calculated_host_listings_count_shared_rooms", Int64.Type}, {"reviews_per_month", type number}}),
    NombresColumnas = Table.ColumnNames(#"Tipo cambiado"),
    NuevosNombres = List.Transform(NombresColumnas, each Text.Proper(_)),
    #"La primera letra mayuscula en nombre columnas" = Table.RenameColumns(#"Tipo cambiado", List.Zip({NombresColumnas, NuevosNombres})),
    #"Columnas con nombre cambiado" = Table.RenameColumns(#"La primera letra mayuscula en nombre columnas",{{"Id", "Id_Listing"}}),
    #"Columnas quitadas" = Table.RemoveColumns(#"Columnas con nombre cambiado",{"Listing_Url", "Scrape_Id", "Last_Scraped", "Space", "Summary"}),
    #"Columnas quitadas1" = Table.RemoveColumns(#"Columnas quitadas",{"Experiences_Offered", "Neighborhood_Overview", "Notes", "Transit", "Access", "Interaction", "House_Rules", "Thumbnail_Url", "Medium_Url", "Picture_Url", "Xl_Picture_Url", "Host_Url", "Host_About", "Host_Thumbnail_Url", "Host_Picture_Url", "Host_Total_Listings_Count", "Host_Verifications"}),
    #"Columnas quitadas2" = Table.RemoveColumns(#"Columnas quitadas1",{"Street", "Neighbourhood"}),
    #"Columnas con nombre cambiado1" = Table.RenameColumns(#"Columnas quitadas2",{{"Neighbourhood_Cleansed", "Neighbourhood"}}),
    #"Columnas quitadas3" = Table.RemoveColumns(#"Columnas con nombre cambiado1",{"Neighbourhood_Group_Cleansed", "State", "Zipcode", "City"}),
    #"Columnas con nombre cambiado2" = Table.RenameColumns(#"Columnas quitadas3",{{"Market", "City"}}),
    #"Borrar columnas irrelevantes" = Table.RemoveColumns(#"Columnas con nombre cambiado2",{"Smart_Location", "Country_Code", "Country", "Square_Feet", "Weekly_Price", "Monthly_Price", "Minimum_Minimum_Nights", "Maximum_Minimum_Nights", "Minimum_Maximum_Nights", "Maximum_Maximum_Nights", "Minimum_Nights_Avg_Ntm", "Maximum_Nights_Avg_Ntm", "Calendar_Updated", "Calendar_Last_Scraped", "Number_Of_Reviews_Ltm", "First_Review", "Last_Review", "Requires_License", "License", "Jurisdiction_Names", "Is_Business_Travel_Ready", "Calculated_Host_Listings_Count", "Calculated_Host_Listings_Count_Entire_Homes", "Calculated_Host_Listings_Count_Private_Rooms", "Calculated_Host_Listings_Count_Shared_Rooms", "Host_Location", "Host_Neighbourhood", "Maximum_Nights", "Minimum_Nights"}),
    #"Poner En Mayúsculas Cada Palabra" = Table.TransformColumns(#"Borrar columnas irrelevantes",{{"Name", Text.Proper, type text}, {"Description", Text.Proper, type text}, {"Host_Name", Text.Proper, type text}, {"Host_Response_Time", Text.Proper, type text}}),
    #"Valor reemplazado" = Table.ReplaceValue(#"Poner En Mayúsculas Cada Palabra","","No Data",Replacer.ReplaceValue,{"Host_Response_Time"}),
    #"Valor reemplazado1" = Table.ReplaceValue(#"Valor reemplazado","N/A","No Data",Replacer.ReplaceText,{"Host_Response_Time"}),
    #"Valor reemplazado2" = Table.ReplaceValue(#"Valor reemplazado1","N/A","No Data",Replacer.ReplaceText,{"Host_Response_Rate"}),
    #"Valores reemplazados con No Data" = Table.ReplaceValue(#"Valor reemplazado2","N/A","No Data",Replacer.ReplaceText,{"Host_Acceptance_Rate"}),
    #"Valor reemplazado3" = Table.ReplaceValue(#"Valores reemplazados con No Data","t","Superhost",Replacer.ReplaceText,{"Host_Is_Superhost"}),
    #"Valor reemplazado4" = Table.ReplaceValue(#"Valor reemplazado3","f","Host",Replacer.ReplaceText,{"Host_Is_Superhost"}),
    #"Valor reemplazado5" = Table.ReplaceValue(#"Valor reemplazado4","t","Yes",Replacer.ReplaceText,{"Host_Has_Profile_Pic"}),
    #"Valor reemplazado6" = Table.ReplaceValue(#"Valor reemplazado5","f","No",Replacer.ReplaceText,{"Host_Has_Profile_Pic"}),
    #"Valor reemplazado7" = Table.ReplaceValue(#"Valor reemplazado6","f","No",Replacer.ReplaceText,{"Host_Identity_Verified"}),
    #"Valor reemplazado8" = Table.ReplaceValue(#"Valor reemplazado7","t","Yes",Replacer.ReplaceText,{"Host_Identity_Verified"}),
    #"Valor reemplazado9" = Table.ReplaceValue(#"Valor reemplazado8","Other (International)","Buenos Aires",Replacer.ReplaceText,{"City"}),
    #"Valor reemplazado10" = Table.ReplaceValue(#"Valor reemplazado9","Mendoza","Buenos Aires",Replacer.ReplaceText,{"City"}),
    #"Valor reemplazado11" = Table.ReplaceValue(#"Valor reemplazado10","Ocean City","Buenos Aires",Replacer.ReplaceText,{"City"}),
    #"Valor reemplazado12" = Table.ReplaceValue(#"Valor reemplazado11","South Florida Gulf Coast","Buenos Aires",Replacer.ReplaceText,{"City"}),
    #"Valor reemplazado13" = Table.ReplaceValue(#"Valor reemplazado12","Beirut","Buenos Aires",Replacer.ReplaceText,{"City"}),
    #"Normalizar Ciudad de Buenos Aires" = Table.ReplaceValue(#"Valor reemplazado13","","Buenos Aires",Replacer.ReplaceValue,{"City"}),
    #"Valor reemplazado14" = Table.ReplaceValue(#"Normalizar Ciudad de Buenos Aires","t","Yes",Replacer.ReplaceText,{"Is_Location_Exact"}),
    #"Valor reemplazado15" = Table.ReplaceValue(#"Valor reemplazado14","f","No",Replacer.ReplaceText,{"Is_Location_Exact"}),
    #"Columnas quitadas4" = Table.RemoveColumns(#"Valor reemplazado15",{"Has_Availability", "Availability_30", "Availability_60", "Availability_90", "Availability_365"}),
    #"Valor reemplazado16" = Table.ReplaceValue(#"Columnas quitadas4","f","NotInstantBookable",Replacer.ReplaceText,{"Instant_Bookable"}),
    #"Valor reemplazado17" = Table.ReplaceValue(#"Valor reemplazado16","t","InstantBookable",Replacer.ReplaceText,{"Instant_Bookable"}),
    #"Columnas quitadas5" = Table.RemoveColumns(#"Valor reemplazado17",{"Require_Guest_Profile_Picture", "Require_Guest_Phone_Verification"}),
    Media = List.Average(#"Columnas quitadas5"[Price]),
    DesviacionEstandar = List.StandardDeviation(#"Columnas quitadas5"[Price]),
    LimiteInferior = Media - 3 * DesviacionEstandar,
    LimiteSuperior = Media + 3 * DesviacionEstandar,
    FilasFiltradas = Table.SelectRows(#"Columnas quitadas5", each [Price] >= LimiteInferior and [Price] <= LimiteSuperior)
in
    FilasFiltradas
```
</details>

<details>
  <summary>Tabla Reviews</summary>

```M
let
    Origen = Csv.Document(File.Contents("C:\Users\Grethel\Documents\Henry\M5 -Data Anlytics\Proyecto Integrador\2 AirBnB\reviews.csv"),[Delimiter=",", Columns=6, Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
    #"Encabezados promovidos" = Table.PromoteHeaders(Origen, [PromoteAllScalars=true]),
    #"Tipo cambiado" = Table.TransformColumnTypes(#"Encabezados promovidos",{{"listing_id", Int64.Type}, {"id", Int64.Type}, {"date", type date}, {"reviewer_id", Int64.Type}, {"reviewer_name", type text}, {"comments", type text}}),
    #"NombresColumnas" = Table.ColumnNames(#"Tipo cambiado"),
    NuevosNombres = List.Transform(NombresColumnas, each Text.Proper(_)),
    RenamedColumns = Table.RenameColumns(#"Tipo cambiado", List.Zip({NombresColumnas, NuevosNombres})),
    #"Columnas con nombre cambiado" = Table.RenameColumns(RenamedColumns,{{"Id", "Id_review"}}),
    #"Texto limpio" = Table.TransformColumns(#"Columnas con nombre cambiado",{{"Comments", Text.Clean, type text}}),
    #"Texto recortado" = Table.TransformColumns(#"Texto limpio",{{"Comments", Text.Trim, type text}})
in
    #"Texto recortado"
```
</details>