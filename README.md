openapi: 3.0.3
info:
  title: CPLX_3FLASH Remote APIs
  version: "1.0.0"
  description: Remote Command, Telemetry, and Synchronization APIs for CPLX_3FLASH.
servers:
  - url: https://api.cplx3flash.example.com
paths:
  /remote/command:
    post:
      summary: Submit remote command
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CommandRequest'
      responses:
        '200':
          description: Job accepted
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommandStatus'
  /remote/command/{job_id}/status:
    get:
      summary: Get command job status
      security:
        - bearerAuth: []
      parameters:
        - name: job_id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Job status
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CommandStatus'
  /telemetry/metrics:
    post:
      summary: Push telemetry metrics
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/MetricPoint'
      responses:
        '200':
          description: Metrics accepted
  /telemetry/metrics:
    get:
      summary: Query telemetry metrics
      security:
        - bearerAuth: []
      parameters:
        - name: since
          in: query
          schema:
            type: string
            format: date-time
      responses:
        '200':
          description: Time-series metrics
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/MetricPoint'
  /telemetry/export:
    post:
      summary: Request telemetry export
      security:
        - bearerAuth: []
      responses:
        '202':
          description: Export scheduled
          content:
            application/json:
              schema:
                type: object
                properties:
                  export_id:
                    type: string
  /sync/start:
    post:
      summary: Start entanglement synchronization session
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/SyncRequest'
      responses:
        '200':
          description: Sync started
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SyncStatus'
  /sync/{sync_id}/health:
    get:
      summary: Get sync session health
      security:
        - bearerAuth: []
      parameters:
        - name: sync_id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Sync health
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SyncStatus'
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
  schemas:
    CommandRequest:
      type: object
      required: [command, target_module]
      properties:
        command:
          type: string
        target_module:
          type: string
        params:
          type: object
    CommandStatus:
      type: object
      properties:
        job_id:
          type: string
        status:
          type: string
        started_at:
          type: string
          format: date-time
        finished_at:
          type: string
          format: date-time
        logs:
          type: string
    MetricPoint:
      type: object
      required: [timestamp, cpu, memory, network]
      properties:
        timestamp:
          type: string
          format: date-time
        cpu:
          type: number
          format: float
        memory:
          type: number
          format: float
        network:
          type: number
          format: float
        queue_len:
          type: integer
    SyncRequest:
      type: object
      required: [peer_id]
      properties:
        peer_id:
          type: string
        channel_params:
          type: object
    SyncStatus:
      type: object
      properties:
        sync_id:
          type: string
        status:
          type: string
        fidelity:
          type: number
          format: float
        started_at:
          type: string
          format: date-time
        last_check:
          type: string
          format: date-time