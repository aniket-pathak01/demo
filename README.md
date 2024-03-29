from rest_framework import status
from rest_framework.response import Response

from sully_engine.ehrbridge.eclinicalworks.transformers.notes import (
    NotesQueryTransformer as NotesQueryPractice,
)


class Notes:
    def __init__(self, data, ehr_id, customer_uuid, ehr_name, **kwargs):
        self.request_body = data
        self.source_data = data.get("body")
        self.ehr_name = ehr_name
        self.ehr_id = ehr_id
        self.customer_id = customer_uuid
        self.kwargs = kwargs

    def new(self):
        event = (
            self.source_data.get("Meta").get("Event")
            if self.source_data.get("Meta").get("Event")
            else ["All"]
        )
        new_note = None
        
        if new_note is None:
            return Response(
                {"detail": "EHR does not exist"}, status=status.HTTP_501_NOT_IMPLEMENTED
            )

        if self.ehr_name == "openemr":
            new_note = new_note.transform(event)
        else:
            new_note = new_note.transform()
        if type(new_note) == Response:
            return new_note

        return Response(new_note)

    def query(self):
        event = (
            self.source_data.get("Meta").get("Event")
            if self.source_data.get("Meta").get("Event")
            else ["All"]
        )

        notes_query = None
        print(self.ehr_name, ">>>>><><<><><>><><>self ehr name")
        if self.ehr_name == "athena":
            notes_query = NotesQueryPractice(
                self.source_data, self.customer_id, self.ehr_id, **self.kwargs
            )
        elif self.ehr_name == "eclinicalworks":

            notes_query = NotesQueryPractice(
                self.source_data, self.customer_id, self.ehr_id, **self.kwargs
            )
        print(">>>>><><<><><>><><> Data1")
        
        if notes_query is None:
            return Response(
                {"detail": "EHR does not exist"}, status=status.HTTP_501_NOT_IMPLEMENTED
            )

        print(">>>>><><<><><>><><> Data")

        if self.ehr_name in ["openemr", "eclinicalworks", "cerner"]:
            notes_query = notes_query.transform()
        else:
            notes_query = notes_query.transform()

        print(">>>>><><<><><>><><> Data2")

        if type(notes_query) == Response:
            print(">>>>>>>>>>>> Return?")
            return notes_query
        
        print(">>>>><><<><><>><><> Data3")

        return Response(notes_query)
